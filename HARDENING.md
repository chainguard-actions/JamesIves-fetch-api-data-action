<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed lib/main.js without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/lib/main.js b/lib/main.js
index 5fb9427..c21555d 100644
--- a/lib/main.js
+++ b/lib/main.js
@@ -1,7 +1,20986 @@
 "use strict";
-var __importDefault = (this && this.__importDefault) || function (mod) {
-    return (mod && mod.__esModule) ? mod : { "default": mod };
+var __getOwnPropNames = Object.getOwnPropertyNames;
+var __commonJS = (cb, mod) => function __require() {
+  try {
+    return mod || (0, cb[__getOwnPropNames(cb)[0]])((mod = { exports: {} }).exports, mod), mod.exports;
+  } catch (e) {
+    throw mod = 0, e;
+  }
+};
+
+// node_modules/@actions/core/lib/utils.js
+var require_utils = __commonJS({
+  "node_modules/@actions/core/lib/utils.js"(exports2) {
+    "use strict";
+    Object.defineProperty(exports2, "__esModule", { value: true });
+    exports2.toCommandValue = toCommandValue;
+    exports2.toCommandProperties = toCommandProperties;
+    function toCommandValue(input) {
+      if (input === null || input === void 0) {
+        return "";
+      } else if (typeof input === "string" || input instanceof String) {
+        return input;
+      }
+      return JSON.stringify(input);
+    }
+    function toCommandProperties(annotationProperties) {
+      if (!Object.keys(annotationProperties).length) {
+        return {};
+      }
+      return {
+        title: annotationProperties.title,
+        file: annotationProperties.file,
+        line: annotationProperties.startLine,
+        endLine: annotationProperties.endLine,
+        col: annotationProperties.startColumn,
+        endColumn: annotationProperties.endColumn
+      };
+    }
+  }
+});
+
+// node_modules/@actions/core/lib/command.js
+var require_command = __commonJS({
+  "node_modules/@actions/core/lib/command.js"(exports2) {
+    "use strict";
+    var __createBinding = exports2 && exports2.__createBinding || (Object.create ? (function(o, m, k, k2) {
+      if (k2 === void 0) k2 = k;
+      var desc = Object.getOwnPropertyDescriptor(m, k);
+      if (!desc || ("get" in desc ? !m.__esModule : desc.writable || desc.configurable)) {
+        desc = { enumerable: true, get: function() {
+          return m[k];
+        } };
+      }
+      Object.defineProperty(o, k2, desc);
+    }) : (function(o, m, k, k2) {
+      if (k2 === void 0) k2 = k;
+      o[k2] = m[k];
+    }));
+    var __setModuleDefault = exports2 && exports2.__setModuleDefault || (Object.create ? (function(o, v) {
+      Object.defineProperty(o, "default", { enumerable: true, value: v });
+    }) : function(o, v) {
+      o["default"] = v;
+    });
+    var __importStar = exports2 && exports2.__importStar || /* @__PURE__ */ (function() {
+      var ownKeys = function(o) {
+        ownKeys = Object.getOwnPropertyNames || function(o2) {
+          var ar = [];
+          for (var k in o2) if (Object.prototype.hasOwnProperty.call(o2, k)) ar[ar.length] = k;
+          return ar;
+        };
+        return ownKeys(o);
+      };
+      return function(mod) {
+        if (mod && mod.__esModule) return mod;
+        var result = {};
+        if (mod != null) {
+          for (var k = ownKeys(mod), i = 0; i < k.length; i++) if (k[i] !== "default") __createBinding(result, mod, k[i]);
+        }
+        __setModuleDefault(result, mod);
+        return result;
+      };
+    })();
+    Object.defineProperty(exports2, "__esModule", { value: true });
+    exports2.issueCommand = issueCommand;
+    exports2.issue = issue;
+    var os = __importStar(require("os"));
+    var utils_1 = require_utils();
+    function issueCommand(command, properties, message) {
+      const cmd = new Command(command, properties, message);
+      process.stdout.write(cmd.toString() + os.EOL);
+    }
+    function issue(name, message = "") {
+      issueCommand(name, {}, message);
+    }
+    var CMD_STRING = "::";
+    var Command = class {
+      constructor(command, properties, message) {
+        if (!command) {
+          command = "missing.command";
+        }
+        this.command = command;
+        this.properties = properties;
+        this.message = message;
+      }
+      toString() {
+        
... (truncated)
```

Locations:

- `lib/main.js`

## Iteration Notes

### Iteration 1

**Notes:**

I could not re-apply the prior-fix patch. All I could see of it was the start of a ~21,000-line esbuild bundle that inlines node_modules (@actions/core etc.) into lib/main.js, and nothing in that visible part is a security change. Rebuilding the bundle by hand isn't possible here, and it isn't needed to run: node_modules is still in the action (@actions/core 2.0.2, @actions/http-client 3.0.1, undici 5.29.0, mustache, async-retry), so the unbundled lib/main.js -> lib/lib.js -> lib/fetch.js chain loads as it should. I left lib/main.js as upstream wrote it.

Because I couldn't recover the lost security change, I added one fix instead, chosen as a likely candidate. It is my own guess, not a copy of the earlier patch. The credentials the token-endpoint request returns were printed to the logs whenever debug is on, and could also leak through the error message. In lib/fetch.js, which is the code that actually runs, and in src/fetch.ts to match, a new maskSecrets() helper now calls core.setSecret on the raw token response and on every non-empty string value inside its JSON. This happens before the error check and before the debug logging. It only changes what shows up in logs: the saved files, environment variables and outputs are the same. Nothing was built or tested. Someone should review whether the earlier bundled harden contained another change that is still missing.

