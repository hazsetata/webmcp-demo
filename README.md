# WebMCP demo

Small Vue 3, TypeScriptTailwindCSS and Vite demo-app that utilizes the new 
[WebMCP](https://developer.chrome.com/blog/webmcp-epp) framework.

To be able to use it, do the following:
 * Install Chrome Beta (146+)
 * Enable WebMCP in `chrome://flags` (search for MCP)
 * Install [Model Context Tool Inspector](https://chromewebstore.google.com/detail/model-context-tool-inspec/gbpdfapgefenggkahomfgkhfehlcenpd) extension 

Build and run the app: `npm run dev` and go to http://localhost:5173/. 
Open the inspector extension and interact with the tools discovered from the page. 
`ADD` tool is alwys available, while `REMOVE` is only available if there are items 
on the list.

