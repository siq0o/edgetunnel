# Add a port map to allow the use of non-standard proxy ports:

```
{
  "80":8080,
  "443":8443
}
```

For example, the above setup will redirect requests originally targeting port 443 to 8443. Similar redirection works for port 80 as well (redirect to proxyIP:8080).

Note: the keys in a JSON must be strings. `"443":8443` is valid but `443:8443` is not.

To use it, you need to set "PORTMAP" environmental to a JSON like the above example, it will only be valid if "PROXYIP" is set.
# Add VLESS outbound

To use it, you need to set "VLESS" environmental to a standard Vless sharing link. **Cloudflare Worker runtime has imposed outbound port restrictions where a TLS (WebSocket secured) outbound request must be made on port 443, and a non-TLS (plain WebSocket) outbound request must be made on port 80. Due to these restrictions, chaining to a VLESS server not running on port 80 (ws) or 443 (wss) won't work.**

See: https://community.cloudflare.com/t/port-forwarding-with-worker/528002
# Support UDP outbound

    1. Direct outbound if run on NodeJS

    2. Forward UDP requests to a remote Vless server if run on CF Worker

    3. Otherwise reject the UDP outbound


# Fallback mechanism

When handling outbound, the program goes through outbounds defined in `globalConfig.outbounds` sequencially. If one fails, the next outbound (if any) will be tried, until it reaches the end of the outbound chain. `globalConfig.outbounds` is similar to the `outbounds` object in a standard ?ray config file, but with some limitations:

    1. `protocol: forward`: our addon, used to forward the outbound TCP traffic directly to a proxy server.

    2. socks and vless only support one user-pass per server.

    3. socks outbound does not support UDP.


If run on a CF Worker, the resultant outbound sequence will be `Direct (via worker), Forward (via PROXYIP, PORTMAP applies), VLESS, SOCKS`, see `setConfigFromEnv()`. If a certain environmental variable is not set, the corresponding outbound will be skipped. Note that Direct outbound will always exist and will be attempted first.
# Abstract worker-vless.js, allow it to be used as a module elsewhere or deploy it locally, see "node/index.js".

It exposes a number of functions and interfaces to be used in an external launcher:

    1. `platformAPI`: defines how to create TCP/Websocket/UDP connection in each platform, required to set if not run on CF Workers.

    2. `globalConfig`: all configurations, such as UUID, outbound methods.

    3. `setConfigFromEnv`: a simplifer way of setting outbound, the caller should pass a JSON which may contain PROXYIP, PORTMAP, VLESS, and SOCKS string fields.

    4. `vlessOverWSHandler(webSocket, earlyDataHeader)`: Process an accepted "webSocket" connection, should be called when a new Websocket connection is established. "webSocket" is a Nodejs WS-compatible object, it must be an accepted one before calling this function. "earlyDataHeader" is a base64 string for ws 0rtt, its value comes from an optional field "sec-websocket-protocol" in the request header.

    5. `getVLESSConfig(hostName)`: Return a human-readable webpage, describing the client config. "getVLESSConfig" now uses the latest active UUID.

    6. `redirectConsoleLog(logServer, instanceId)`: Call this function to mirror `console.log` and POST it to a HTTP(s) server, set `LOGPOST` environmental variable in CF Worker to use this function. `logServer` is the URL, `instanceId` can be an UUID or a random number.


Using the provided wrapper to run in NodeJS only requires the ws library (see "node/setup.sh"), no need to install wrangler or other dependencies.

