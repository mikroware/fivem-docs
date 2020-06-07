---
title: Fullscreen NUI
weight: 441
---

The most common use case of NUI is a full-screen 'UI page', which is overlaid on top of the game and may or may not have
input focus. These are supported on both FiveM and RedM at this time, and are part of basic CitizenFX framework level
support.

The following natives are related to using full-screen NUI:

* {{<native_link "SEND_NUI_MESSAGE">}}
* {{<native_link "SET_NUI_FOCUS">}}

Using any of the above natives will load the NUI frame. Messages send to the NUI will be buffered till the first listener is added (globally for all resources).

## Setting up a fullscreen NUI page
To assign a full-screen NUI page to a resource, currently you need to specify a single `ui_page` in the
[resource manifest][resource-manifest] for the resource containing an UI page, like shown below:

```lua
-- specify the root page, relative to the resource
ui_page 'main.html'

-- every client-side file still needs to be added to the resource packfile!
files {
    'main.html'
}
```

## Referencing other assets
{{% alert title="Warning" color="warning" %}}
Note that absolute NUI asset references **require** your resource name to be lowercased! This is due to DNS name
restrictions.
{{% /alert %}}

The NUI system registers a `nui://` protocol scope for resource files. Therefore, you can reference a file in a resource
as follows:

```html
<script type="text/javascript" src="nui://my-resource/production.js" async></script>
```

This also means you can use the Chromium developer tools to, say, fetch _any_ packaged resource file (including client
scripts) simply using `fetch('nui://spawnmanager/fxmanifest.lua')` or similar in the developer console. Open source for
everyone! Anyone trying to sell you ways to 'dump assets' has basically been scamming you.

<!-- #GAMETODO: block this? but then we'll get NUI bypasses.. eww -->

## Developer tools
CEF remote debugging tools are exposed on [http://localhost:13172/](http://localhost:13172/) as long as the game is
running. You can use any Chromium-based browser to easily access these tools.

Alternately, it can be opened using the `nui_devTools` command in the game's <kbd>F8</kbd> console.

## NUI focus
There's a limited focus stack for NUI resources, you can set focus to the **current** resource using the
{{<native_link "SET_NUI_FOCUS">}} native, which will set keyboard focus and/or mouse cursor focus depending on the
provided arguments.

The most recently focused resource will be ordered on top of the focus stack, and resources are currently implemented
as full-screen iframes: that means there's no click-through across resources.

## NUI messages
You can send a [message][mdn-messages] to the current resource's NUI page using the {{<native_link "SEND_NUI_MESSAGE">}}
native, or if using Lua, the convenience wrapper [SendNUIMessage][send-nui-message] which encodes a JSON string for you.

For example:

```lua
-- Lua
SendNUIMessage({
    type = 'open'
})
```

```js
// JS
SendNuiMessage(JSON.stringify({
    type: 'open'
}))
```

```csharp
// C#, assumes Newtonsoft.Json PCL version is referenced
SendNuiMessage(JsonConvert.SerializeObject(new
{
    type = "open"
}));
```

```js
// browser side
window.addEventListener('message', (event) => {
    if (event.data.type === 'open') {
        doOpen();
    }
});
```

[mdn-messages]: https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage#The_dispatched_event
[send-nui-message]: /docs/scripting-reference/runtimes/lua/functions/SendNUIMessage
[resource-manifest]: /docs/scripting-reference/resource-manifest/resource-manifest
