# General Guide on SFMC developing

- [General Guide on SFMC developing](#General-Guide-on-SFMC-developing)
	- [Development Environment](#Development-Environment)
		- [Use Live Linting & Auto-Formatting](#Use-Live-Linting--Auto-Formatting)
		- [Use Syntax Highlighting](#Use-Syntax-Highlighting)
		- [Where to code: GUI vs. online editor vs. IDE](#Where-to-code-GUI-vs-online-editor-vs-IDE)
		- [How to debug quickly](#How-to-debug-quickly)
		- [Online Services for quick debugging](#Online-Services-for-quick-debugging)
	- [Project Setup](#Project-Setup)
		- [Folder structure](#Folder-structure)
		- [File structure](#File-structure)
	- [Data Extensions](#Data-Extensions)
		- [SEO, Page title](#SEO-Page-title)
		- [Data types](#Data-types)
		- [updating External key of Data Extensions](#updating-External-key-of-Data-Extensions)

## Development Environment

### Use Live Linting & Auto-Formatting

Now, this unfortunately is NOT yet available for AMPscript but one day in the future... well either AMPscript is gone or somebody will have written a plugin. Aside from that, for HTML, CSS, JavaScript and whatever preprocessor you might want to use in your project, there are powerful tools available that can auto-fix minor code issues and enforce code formatting rules.

_Why?_ Linter's will tell you about potential code problems before they are published to PROD, making your life a hell lot easier.

_Why?_ Having to adapt to multiple styles in one project makes the code harder to read and more difficult to debug.

_Why?_ Your client will likely ask you to adhere to certain standards. Adapting your default configuration to these client standards usually reduces time spend on code reviews to the bare minimum.

_Why?_ Junior developers can learn from how their code is changed by the auto-formatters and

_Why?_ Still not convinced? The guys over at Prettier wrote an actual [essay](https://prettier.io/docs/en/why-prettier.html) on this.

### Use Syntax Highlighting

For Visual Studio Code you can get this [AMPscript syntax highlighter](https://marketplace.visualstudio.com/items?itemName=sergey-agadzhanov.AMPscript). Also, you should manually assign `*.ssjs` of type JavaScript, providing you more options when it comes to linting and auto-formatting than if you were using `*.js`. It also logically separates your SSJS from JS, avoiding confusion.
Checkout the [.eslintrc](../.eslintrc) and [.prettierrc](../.prettierrc) which includes a configuration for JS, CSS and SSJS.

### Where to code: GUI vs. online editor vs. IDE

By all means, avoid making changes directly in SFMC's GUI. Best practice is to make all your changes in your local IDE instead (I recommend the free [Visual Studio Code](https://code.visualstudio.com/)).

_Why?_ Whenever you save a content block, there is no going back. No CTRL+Z, no undo last change - nothing. Your previous code is imply gone.

_Why?_ If others open your CloudPage and accidentally make changes you (a) will never find out who did it and (b) have no way of recovering the previous version.

_Why?_ The GUI does not offer proper code highlighting (even if you wrap your code in `<script>...</script>`)

_Why?_ The GUI does not offer any linting / error correction. In your IDE you can utilize [ESLint](https://eslint.org/) for your SSJS and JS code, as well as [Prettier](https://prettier.io/) for your CSS

### How to debug quickly

Ever tried to fix a bug and got annoyed by having to publish your page or send your email 1000 times per day? Turns out you can load **any** text file that’s freely available on the internet and run it through the server-side engines for both AmpScript and SSJS. The key is a method that’s similar to JavaScript’s `eval()` method together with another method that grabs the text content of a URL and returns it as a String:

```javascript
/* put this into an HTML block in your debug CloudPage (nothing else) */
%%= TreatAsContent(HTTPGet("https://myurl.com/loader.html")) =%%
```

```html
<!-- loader.html -->

<!-- optionally load AmpScript file -->
%%= TreatAsContent(HTTPGet("https://myurl.com/myampcode.amp")) =%%

<script runat="server" language="JavaScript">

Platform.Load("core", "1.1.5"); // define one core version up front for all

// load SSJS file #1 via AmpScript
%%= TreatAsContent(HTTPGet("https://myurl.com/mycode1.ssjs")) =%%
// load SSJS file #2 via AmpScript
%%= TreatAsContent(HTTPGet("https://myurl.com/mycode2.ssjs")) =%%
// load SSJS file #3 via AmpScript
%%= TreatAsContent(HTTPGet("https://myurl.com/mycode3.ssjs")) =%%

</script>

<!-- optionally load HTML file via AmpScript -->
%%= TreatAsContent(HTTPGet("https://myurl.com/app.html")) =%%
```

```javascript
// mycode1.ssjs
function MyUtilities() {
    ...
}
```

```javascript
// mycode2.ssjs
function MySfmcApiWrapper() {
    ...
}
```

```javascript
// mycode3.ssjs
var util = MyUtilities();
var sfmc = MySfmcApiWrapper();

// ... fancy app code here
```

_Why?_ You save time because now you only update your online file on whatever webserver you like and don't need to re-publish your debug page ever again.

_How?_ Internally, AmpScript is always processed & executed first and only afterwards it runs through the SSJS processor.

_Thanks_ to my buddy Christian who published a [full tutorial](https://simple-force.com/2018/12/17/marketing-cloud-best-practice-external-editors/) a while ago and taught me how to use this hack.

### Online Services for quick debugging

If you have access to your own web-server it is definitely recommended to use that one rather than a free online service for security reasons. Also, being able to integrate via SFTP will speed up your process.

If you do not have that, there are multiple services out there that offer quick and easy access to files you created online. The easiest services for this purpose at the time of writing seem to be c9.io or codesandbox.io.

**Note:** They might automatically apply formatting that breaks your code at their own will when you paste code there, making it imperative to use `*.ssjs` and `*.amp` file extensions as these will not be recognized and hence not auto-formatted.

**Important:** Please keep in mind never to store credentials in these online services to avoid security issues! If credentials or secrets are needed, retrieve them from SFMC's Key management (Administration section; for `De-/EncryptSymmetric()`) or from a Data Extensions if you need to use other types of credentials.

## Project Setup

### Folder structure

This is by far up to personal taste but I would recommend to stick any one folder structure in your Git repos for every CloudPage/E-Mail. For CloudPages and e-Mails a similar structure can be used, but keep in mind that JavaScript (app/) will not be executed in e-Mails and SFMC actually automatically removes any non-SSJS `<script>` nodes from your content blocks without warning.

This is what I would recommend after playing around with it for some time:

-   project-root/
    -   cloudPages/
        -   cloudPageName1/
            -   app/ _(this is where all front end code goes)_
                -   lib/
                    -   jquery.min.js
                    -   bootstrap.min.css
                    -   ... _(optional; libraries that you want to host in SFMC)_
                -   scss/
                    -   ... _(optional; all your SASS files)_
                -   js/
                    -   ... _(all your non-minified JS files)_
                -   app-1.html
                -   app-2.loadExternalLibs.html
                -   app-3.style.min.css
                -   app-4.script.min.js
            -   server/ _(this is where SSJS and AMPscript goes)_
                -   lib/
                    -   ... _(any SSJS files containing shared classes / polyfills, ...)_
                -   server-1.amp
                -   server-2.initCore.ssjs
                -   server-3.ssjs
        -   cloudPageName2/
            -   ...
    -   emails/
        -   ...
    -   contentBlocks/ _(sometimes you will want to prepare code snippets)_
        -   snippet1/
        -   ...

How you structure your `app/` directory is likely up to whatever front end library you might be using. Angular(JS), react and others all come with their standard directory approach and it is recommended to follow these best practices.

In any case, you should try to map your local files to one content block each in the CloudPage - except for when you minify your front end code, then one content block per code type / per .min file should be used.

```html
<!-- model layout of a CloudPage -->

<!-- Start of server-side code here -->
<!-- block 1 -->
server-1.amp

<!-- block 2 -->
server-2.initCore.ssjs

<!-- block 3 -->
lib.mySharedClass.ssjs

<!-- block 4 -->
server-3.ssjs
<!-- END of server-side code here -->

<!-- block 5 - this is where all your HTML goes -->
app-1.html

<!-- start loading CSS & JS below here -->
<!-- block 6 -->
app-2.loadExternalLibs.html

<!-- block 7 -->
app-3.style.min.css

<!-- block 8 -->
app-4.script.min.js
```

Load the Platform Core once before all other SSJS files to use one consistent version across the CloudPage:

```html
<!-- initCore.ssjs -->
<!-- load this before all other SSJS files to use one consistent Core version across the CloudPage -->
%%[ /* <b>initCore.ssjs</b> */ ]%%
<script runat="server" language="JavaScript">
	Platform.Load('core', '1.1.5'); // choose a version suitable to your needs!
</script>
```

Keep in mind that the above is just the minimal example. Of you have more code, it's good to split it up into files of up to 150 lines each, if possible.

**Important:** For optimal code highlighting, linting and auto-formatting you will need to remove the `<script>` ndoes around SSJS and `<style>` nodes around CSS in your local `*.ssjs` & `*.css` files. In the online version on the SFMC server however, you do need to include these in every file.

### File structure

Any file should only contain markup language (HTML) or front-end code (CSS or JS) or AMPscript code or SSJS. You should avoid mixing this and mimic what the structure you saved locally.
To still be able to easily recognize which content block contains what, add a little AMPscript at the beginning:

```java
%%[ /* myAMPscript.amp */ ]%%

%%[
// insert AMPscript code here
]%%
```

or

```java
%%[ /* mySSJS.ssjs */ ]%%

<script runat="server" language="JavaScript">
// insert SSJS code here
</script>
```

## Data Extensions

They can be set up via Email Studio and via Audience Builder ("Contact Builder"). The two interfaces look mostly similar but they do have distinct differences:

| Feature                             | Email Studio                                                                                                                                                                                                                                                                                                                                                                                                                                        | Audience/Contact Builder                                         |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Where to find DEs                   | top navigation > Subscribers > Data Extensions                                                                                                                                                                                                                                                                                                                                                                                                      | top navigation > Data Extensions                                 |
| Sharing with BUs                    | You can either share an entire folder or a single DE.<br> - Sharing a folder: via the right-click menu of shared folders in the left navigation.<br> - Sharing single DE: Open the DE, then click the Permissions tab. The "Sharing Status" column reveals the sharing status for each BU.<br> - You can distinctly choose what access rights are granted: `View`, `Update`, `Delete`, `Manage Data`, `Transfer Ownership`, `Manage Data Retention` | n/a                                                              |
| Data Import sources                 | FTP, Upload (including Zips)                                                                                                                                                                                                                                                                                                                                                                                                                        | FTP, Upload, DataExtensions (no Zips)                            |
| Data Export destinations            | FTP                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Browser download, "Salesforce Objects & Reports", "Enhanced FTP" |
| Single Record Add/Update/Delete     | no                                                                                                                                                                                                                                                                                                                                                                                                                                                  | yes                                                              |
| Delete All Records (Truncate table) | "Clear Data"-button                                                                                                                                                                                                                                                                                                                                                                                                                                 | "Clear Records"-button                                           |
| Columns Add/Update/Delete           | yes, all at once (click on "Edit Fields" to activate); <br>**Note:** this feature is deactivated in Email Studio if you set up data relationships with this DE in Contact Builder!                                                                                                                                                                                                                                                                  | yes, 1-by-1 (even when data relationsships have been set up)     |
| Columns Update - Limitations        | - You may never decrease the length of a field.<br>- You may never change the field type.<br>- If the DE is part of a relationship, you may not delete any field<br>- Decimal length can never be changed                                                                                                                                                                                                                                           |

### SEO, Page title

You can actually easily set the Page Title and a few other SEO relevant settings ususally found in the page header's code via GUI in SFMC. While you are in the folder view, find the burger icon and delete icon on every page. Clicking the burger icon ("Page Properties") brings you to a page with an "Advanced Settings"-button and there to the "SEO" tab. Here you can set up page title, keywords, descriptions, and if search engines should be allowed to index the particular CloudPage.

### Data types

| Name          | SQL equivalent | Max Length                                                     | Example                                                                                                         |
| ------------- | -------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Text          | String         | `4000`                                                         | `"any text."`                                                                                                   |
| Number        | Integer        | ???                                                            | `21`                                                                                                            |
| Date          | ???            | ???                                                            | ???                                                                                                             |
| Boolean       | Boolean        | -                                                              | `True`, `False`                                                                                                 |
| Email Address | String         | `254`                                                          | `"demo@domain.com"`<br>(a filter is applied making sure only standard compliant emails are stored)              |
| Phone         | String         | ???                                                            | ??? (adhering to [E.164 standard](https://en.wikipedia.org/wiki/E.164))                                         |
| Decimal       | Decimal        | `21,17`, max sum is `38`, with max `17` digits after the comma | `1234.56789`                                                                                                    |
| Locale        | Enum           | `5`                                                            | `"en-us"`, [complete list](https://help.salesforce.com/articleView?id=mc_moc_data_extension_locales.htm&type=5) |

### updating External key of Data Extensions

Only works in Email Studio - not in Contact Builder
