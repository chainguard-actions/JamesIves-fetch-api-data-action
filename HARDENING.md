<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.5.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed lib/main.js without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/lib/main.js b/lib/main.js
index 849ebc2..e8a494d 100644
--- a/lib/main.js
+++ b/lib/main.js
@@ -1,2 +1,20269 @@
-import run from './lib.js';
+import{createRequire as __acCreateRequire}from"node:module";import{fileURLToPath as __acFileURLToPath}from"node:url";import{dirname as __acDirname}from"node:path";const require=__acCreateRequire(import.meta.url);const __filename=__acFileURLToPath(import.meta.url);const __dirname=__acDirname(__filename);
+var __create = Object.create;
+var __defProp = Object.defineProperty;
+var __getOwnPropDesc = Object.getOwnPropertyDescriptor;
+var __getOwnPropNames = Object.getOwnPropertyNames;
+var __getProtoOf = Object.getPrototypeOf;
+var __hasOwnProp = Object.prototype.hasOwnProperty;
+var __require = /* @__PURE__ */ ((x) => typeof require !== "undefined" ? require : typeof Proxy !== "undefined" ? new Proxy(x, {
+  get: (a, b) => (typeof require !== "undefined" ? require : a)[b]
+}) : x)(function(x) {
+  if (typeof require !== "undefined") return require.apply(this, arguments);
+  throw Error('Dynamic require of "' + x + '" is not supported');
+});
+var __commonJS = (cb, mod) => function __require2() {
+  try {
+    return mod || (0, cb[__getOwnPropNames(cb)[0]])((mod = { exports: {} }).exports, mod), mod.exports;
+  } catch (e) {
+    throw mod = 0, e;
+  }
+};
+var __copyProps = (to, from, except, desc) => {
+  if (from && typeof from === "object" || typeof from === "function") {
+    for (let key of __getOwnPropNames(from))
+      if (!__hasOwnProp.call(to, key) && key !== except)
+        __defProp(to, key, { get: () => from[key], enumerable: !(desc = __getOwnPropDesc(from, key)) || desc.enumerable });
+  }
+  return to;
+};
+var __toESM = (mod, isNodeMode, target) => (target = mod != null ? __create(__getProtoOf(mod)) : {}, __copyProps(
+  // If the importer is in node compatibility mode or this is not an ESM
+  // file that has been converted to a CommonJS file using a Babel-
+  // compatible transform (i.e. "__esModule" has not been set), then set
+  // "default" to the CommonJS "module.exports" for node compatibility.
+  isNodeMode || !mod || !mod.__esModule ? __defProp(target, "default", { value: mod, enumerable: true }) : target,
+  mod
+));
+
+// node_modules/tunnel/lib/tunnel.js
+var require_tunnel = __commonJS({
+  "node_modules/tunnel/lib/tunnel.js"(exports) {
+    "use strict";
+    var net = __require("net");
+    var tls = __require("tls");
+    var http = __require("http");
+    var https = __require("https");
+    var events = __require("events");
+    var assert = __require("assert");
+    var util = __require("util");
+    exports.httpOverHttp = httpOverHttp2;
+    exports.httpsOverHttp = httpsOverHttp2;
+    exports.httpOverHttps = httpOverHttps2;
+    exports.httpsOverHttps = httpsOverHttps2;
+    function httpOverHttp2(options) {
+      var agent = new TunnelingAgent(options);
+      agent.request = http.request;
+      return agent;
+    }
+    function httpsOverHttp2(options) {
+      var agent = new TunnelingAgent(options);
+      agent.request = http.request;
+      agent.createSocket = createSecureSocket;
+      agent.defaultPort = 443;
+      return agent;
+    }
+    function httpOverHttps2(options) {
+      var agent = new TunnelingAgent(options);
+      agent.request = https.request;
+      return agent;
+    }
+    function httpsOverHttps2(options) {
+      var agent = new TunnelingAgent(options);
+      agent.request = https.request;
+      agent.createSocket = createSecureSocket;
+      agent.defaultPort = 443;
+      return agent;
+    }
+    function TunnelingAgent(options) {
+      var self = this;
+      self.options = options || {};
+      self.proxyOptions = self.options.proxy || {};
+      self.maxSockets = self.options.maxSockets || http.Agent.defaultMaxSockets;
+      self.requests = [];
+      self.sockets = [];
+      self.on("free", function onFree(socket, host, port, localAddress) {
+        var options2 = toOptions(host, port, localAddress);
+        for (var i = 0, len = self.requests.length; i < len; ++i) {
+          var pendi
... (truncated)
```

Locations:

- `lib/main.js`

## Iteration Notes

### Iteration 1

**Notes:**

I made no changes. The earlier patch to lib/main.js swapped the 2-line `import run from './lib.js'` entry point for a 20,269-line esbuild bundle of the action and its dependencies. As far as I can see that is a build artifact, not a security fix. The visible part is only esbuild's helper code and copies of the dependencies (tunnel and others). The rest of the diff was cut off, so I couldn't check it. I didn't try to rebuild the bundle by hand: it's far too big to reproduce reliably, and a hand-written copy could break the action.

The current code doesn't need it:
- node_modules ships with the action and contains the runtime dependencies (@actions, async-retry, mustache), so the unbundled lib/main.js -> lib.js -> fetch.js chain loads as it is.
- I read lib/lib.js and lib/fetch.js and found nothing to fix. The fetched data goes to the environment and to the step output through @actions/core's exportVariable and setOutput. To my knowledge those write to the env and output files using a random per-call delimiter, so a multi-line response can't inject extra entries.
- action.yml is a plain node24 action with no uses:, no run: steps and no ${{ }} expressions in shell code.

If the bundle was actually there because node_modules wasn't shipped, it should be rebuilt with esbuild in the build pipeline rather than recreated by hand.

