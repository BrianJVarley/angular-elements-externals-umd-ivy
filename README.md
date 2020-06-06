# Angular + Ivy + webpack externals/UMD

- Serving: `ng serve -o`
- Building with externals: npm run `build:externals`
- Includes polyfills for web components in IE 11

## Findings

1. All dependencies are part of the `scripts` section within `angular.json`. No need to copy around UMD bundles anymore.
2. To make it work with Ivy, I call ngcc without parameters in the `postinstall` script. The original parameters prevent dealing with UMD.

## How this works:

1. Configure external dependencies you want the web component to inherit from it's host. In this config file `webpack.external.js` you are telling the builder to exclude the following UMD bundles from the web component build.

```JavaScript
module.exports = {
    "externals": {
        //"rxjs": "rxjs", // Tells builder not to exclude this UMD bundle from the build
        "@angular/core": "ng.core", // Tells builder these packages will be inherited from the host app.
        "@angular/common": "ng.common",
        "@angular/common/http": "ng.common.http",
        "@angular/platform-browser": "ng.platformBrowser",
        "@angular/platform-browser-dynamic": "ng.platformBrowserDynamic",
        "@angular/compiler": "ng.compiler",
        "@angular/elements": "ng.elements",
        "@angular/router": "ng.router",
        "@angular/forms": "ng.forms"
    }
}

```

2. Specify scripts to include in the host's globals. This means our host app will expose these UMD bundles specifed
   in the scripts section of `angular.json` project configuration file. See [scripts](https://github.com/angular/angular-cli/wiki/stories-global-scripts#global-scripts):

```JavaScript
	"scripts": [
							{
								"bundleName": "polyfill-webcomp-es5",
								"input": "node_modules/@webcomponents/webcomponentsjs/custom-elements-es5-adapter.js"
							},
							{
								"bundleName": "polyfill-webcomp",
								"input": "node_modules/@webcomponents/webcomponentsjs/bundles/webcomponents-sd-ce-pf.js"
							},
							//"node_modules/rxjs/bundles/rxjs.umd.js", // excluding rxjs from global scripts
							"node_modules/@angular/core/bundles/core.umd.js",
							"node_modules/@angular/common/bundles/common.umd.js",
							"node_modules/@angular/common/bundles/common-http.umd.js",
							"node_modules/@angular/compiler/bundles/compiler.umd.js",
							"node_modules/@angular/elements/bundles/elements.umd.js",
							"node_modules/@angular/platform-browser/bundles/platform-browser.umd.js",
							"node_modules/@angular/platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js"
						]
					},

```

3. Running `npm run build:externals` tells Angular to read the _extra_ `webpack.external.js` config. In this case telling
   the builder to exclude a set of UMD bundles from the web component. Because they will be exposed on the host as globals.
