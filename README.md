Theme similar to Darcula for slack.

Place the following in `/Applications/Slack.app/Contents/Resources/app.asar.unpacked/src/static/ssb-interop.js`

```
// First make sure the wrapper app is loaded
const gitHubRawDomain = "https://raw.githubusercontent.com"
const gitHubBranch = "master"
const gitHubUser = "floralvikings"
const gitHubRepo = "twenty-one"
const customStyles = true

if(customStyles) {
    document.addEventListener("DOMContentLoaded", async function () {

        let customCss = await getAllCss()

        // Then get its webviews
        let webviews = document.querySelectorAll(".TeamView webview");

        // Insert a style tag into the wrapper view
        let s = document.createElement('style');
        s.type = 'text/css';

        s.innerHTML = customCss;
        document.head.appendChild(s);

        // Wait for each webview to load
        webviews.forEach(webview => {
            webview.addEventListener('ipc-message', message => {
                if (message.channel == 'didFinishLoading') {
                    // Finally add the CSS into the webview
                    let script = `
                     let s = document.createElement('style');
                     s.type = 'text/css';
                     s.id = 'slack-custom-css';
                     s.innerHTML = \`${customCss}\`;
                     document.head.appendChild(s);
                     `
                    webview.executeJavaScript(script);
                }
            });
        });
    });
}

async function getAllCss() {
    let styleListUrl = `${gitHubRawDomain}/${gitHubUser}/${gitHubRepo}/${gitHubBranch}/styles.json`;

    let styleListResponse = await fetch(styleListUrl)
    let cssList = await styleListResponse.json()

    let customCss = ""

    for (let i = 0; i < cssList.length; i++) {
        let currentStyleFile = cssList[i];
        let currentStyleUrl = `${gitHubRawDomain}/${gitHubUser}/${gitHubRepo}/${gitHubBranch}/${currentStyleFile}`

        console.log(currentStyleUrl)

        let currentStyleResponse = await fetch(currentStyleUrl)
        let currentStyleText = await currentStyleResponse.text()

        customCss += currentStyleText
        customCss += "\n"
    }

    return customCss
}

```

