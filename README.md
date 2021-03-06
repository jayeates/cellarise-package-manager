# cellarise-package-manager
[![view on npm](http://img.shields.io/npm/v/cellarise-package-manager.svg?style=flat)](https://www.npmjs.org/package/cellarise-package-manager)
[![npm module downloads per month](http://img.shields.io/npm/dm/cellarise-package-manager.svg?style=flat)](https://www.npmjs.org/package/cellarise-package-manager)
[![Dependency status](https://david-dm.org/Cellarise/cellarise-package-manager.svg?style=flat)](https://david-dm.org/Cellarise/cellarise-package-manager)
[![Build Status](https://travis-ci.org/Cellarise/cellarise-package-manager.svg?branch=master)](https://travis-ci.org/Cellarise/cellarise-package-manager)
[![Code
Climate](https://codeclimate.com/github/Cellarise/cellarise-package-manager/badges/gpa.svg)](https://codeclimate.com/github/Cellarise/cellarise-package-manager)
[![Test Coverage](https://codeclimate.com/github/Cellarise/cellarise-package-manager/badges/coverage.svg)](https://codeclimate.com/github/Cellarise/cellarise-package-manager/badges/coverage.svg)

> A custom package manager providing a set of gulp build tasks and a windows command line wrapper around the npm, gulp, and git commands.
## Installation

Install as a global package to enable commandline execution of `cpm`, `cgulp`, `cgitsh`, and `oauth` from any location.

```cmd
npm install -g cellarise-package-manager
```

### Pre-requisites

  * Windows 7/8
    * Unix has not been tested and for this reason is unsupported.
  * [GITWin32 OpenSSL v1.0.1g](http://slproweb.com/products/Win32OpenSSL.html).
    * When installing OpenSSL, you must tell it to put DLLs in the Windows system directory
  * Node packages (global):
    * node-gyp
        * Python ([windows-python-v2.7.3](http://www.python.org/download/releases/2.7.3#download) recommended, `v3.x.x` is __*not*__ supported)
        * Microsoft Visual Studio 2013 for Windows Desktop (Express version works well)
    * gulp
  * Git
    * cygwin is recommended for Windows and use with Atlassian Bamboo.  Install packages git-completion, git-gui, gitk, and openssh.
  * Atlassian suite - for build tasks integrating with Atlassian JIRA or Atlassian Bamboo.

Only 32bit versions of the following pre-requisites have been tested.

### Oauth config

The `oauth` command runs the `oauth-rest-atlassian` package -an OAuth wrapper to authenticate and use the Atlassian REST API. The initial authorisation dance is managed through a local web page.  Follow the instructions from the [package readme](https://www.npmjs.org/package/oauth-rest-atlassian) to create a config.json and private key.  Save these files to the root of this package.



## Usage

### From the command line

The `cpm` command accepts two arguments - the name of the cpm function to be executed and a parameter to pass to the function.

```cmd
cpm install myModule
```

The `cgulp` command accepts one argument - the name of the gulp task to be executed.

```cmd
cgulp test
```

The `cgitsh` command accepts one argument - the name of the bash script to be executed.

```cmd
git-bamboo Git-clone
```

The `oauth` command does not require any arguments.  However, a config.json and a private key must be available in the root of this package.

```cmd
oauth
```

Each bash script uses Atlassian Bamboo build variables.  These variables can be set manually if calling the script outside of Atlassian Bamboo or if you want to override the variable.  For example, to override the build repository:

```cmd
set bamboo_planRepository_repositoryUrl="git@github.com:Cellarise/script-git-bamboo.git"
git-bamboo Git-clone
```


# cpm functions

cpm provides a wrapper around npm commands.

## gulp

Execute the `cgulp` command. First remove any build folders created by a previous execution of `cgulp`. The second argument past to `cpm` is used as the task to be executed.

```bat
REM delete Build, Reports, and Temp folder contents
rmdir .\Build /s /q
rmdir .\Reports /s /q
rmdir .\Temp /s /q
call cgulp %2
```

## clean

Remove build folders created by `cgulp`.

```bat
rmdir .\Build /s /q
rmdir .\Reports /s /q
rmdir .\Temp /s /q
```

## install

Install modules (production dependencies only). First prune any modules no longer in package.json and after installation deduplicate. This function uses the second argument passed to `cpm` as the module name to install.

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm prune
call npm install --msvs_version=2013 --production %2
call npm dedupe --msvs_version=2013
```

## publish

Publish to npm

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm publish
```

## update

Update modules (production dependencies only). First prune any modules no longer in package.json and after installation deduplicate. This function uses the second argument passed to `cpm` as the module name to update.

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm prune
call npm update --msvs_version=2013 --production %2
call npm dedupe --msvs_version=2013
```


# cgulp functions

cgulp enables execution of global tasks defined in this package. See the API for a definition of the avaialble tasks.



# cgitsh functions

## Git-clone

Clone the first repository linked to the bamboo build plan. Clone into folder Git.

```sh
git clone "$bamboo_planRepository_repositoryUrl" Git
exit
```

## Git-release-version

Clone the first repository linked to the bamboo build plan. Clone into folder Git. Then create a new branch to track the change and tag it with the Atlassian Bamboo Jira version. Finally merge the release branch into the production branch.

```sh
git clone "$bamboo_planRepository_repositoryUrl" Git
cd Git
git checkout master
git checkout -b "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version"
git push origin "release/$bamboo_jira_version" --tags
git checkout production
git merge "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version" production
git push origin production --tags
exit
```

## Github-clone

Clone repository from github using the Atlassian Bamboo Jira project name. Clone into folder Build.

```sh
git clone git@github.com:"$bamboo_jira_projectName".git Build
exit
```

## Github-release-version

Stage all changes, commit, and push to the master branch.  Then create a new branch to track the change and tag it with the Atlassian Bamboo Jira version.

```sh
git rm . -r --cached
git add .
git status
git commit -m "Deploy release $bamboo_jira_version"
git push origin master
git checkout -b "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version"
git push -u origin "release/$bamboo_jira_version" --tags
exit
```

## API
### Modules
<dl>
<dt><a href="#module_reports/autoUpdateMocha">reports/autoUpdateMocha</a> ⇒ <code>Object</code></dt>
<dd><p>Auto update mocha report</p>
</dd>
<dt><a href="#module_reports/baseCucumber">reports/baseCucumber</a> ⇒ <code>Object</code></dt>
<dd><p>Base methods for cucumber reports</p>
</dd>
<dt><a href="#module_reports/baseMocha">reports/baseMocha</a> ⇒ <code>Object</code></dt>
<dd><p>Base methods for mocha reports</p>
</dd>
<dt><a href="#module_reports/davidCucumber">reports/davidCucumber</a> ⇒ <code>Object</code></dt>
<dd><p>David cucumber report</p>
</dd>
<dt><a href="#module_reports/davidMocha">reports/davidMocha</a> ⇒ <code>Object</code></dt>
<dd><p>David mocha report</p>
</dd>
<dt><a href="#module_reports/eslintBunyan">reports/eslintBunyan</a> ⇒ <code>String</code></dt>
<dd><p>Eslint bunyan report logger.</p>
</dd>
<dt><a href="#module_reports/eslintCucumber">reports/eslintCucumber</a> ⇒ <code>Object</code></dt>
<dd><p>Eslint cucumber report</p>
</dd>
<dt><a href="#module_reports/eslintMocha">reports/eslintMocha</a> ⇒ <code>Object</code></dt>
<dd><p>Eslint mocha report</p>
</dd>
<dt><a href="#module_reports/genericCucumber">reports/genericCucumber</a> ⇒ <code>Object</code></dt>
<dd><p>Generic cucumber report</p>
</dd>
<dt><a href="#module_reports/genericMocha">reports/genericMocha</a> ⇒ <code>Object</code></dt>
<dd><p>Generic mocha report</p>
</dd>
<dt><a href="#module_reports/postBuildCucumber">reports/postBuildCucumber</a> ⇒ <code>Object</code></dt>
<dd><p>Post build cucumber report</p>
</dd>
<dt><a href="#module_reports/postBuildMocha">reports/postBuildMocha</a> ⇒ <code>Object</code></dt>
<dd><p>Post build mocha report</p>
</dd>
<dt><a href="#module_utils/autoUpdate">utils/autoUpdate</a> ⇒ <code>Object</code></dt>
<dd><p>Auto update build utilities.</p>
</dd>
<dt><a href="#module_utils/bamboo">utils/bamboo</a> ⇒ <code>Object</code></dt>
<dd><p>Bamboo build utilities</p>
</dd>
<dt><a href="#module_utils/config">utils/config</a> ⇒ <code>Object</code></dt>
<dd><p>Application configuration utility</p>
</dd>
<dt><a href="#module_utils/david">utils/david</a> ⇒ <code>Object</code></dt>
<dd><p>David build utilities</p>
</dd>
<dt><a href="#module_utils/jira">utils/jira</a> ⇒ <code>Object</code></dt>
<dd><p>JIRA build utilities</p>
</dd>
<dt><a href="#module_utils/postBuild">utils/postBuild</a> ⇒ <code>Object</code></dt>
<dd><p>Post build utilities</p>
</dd>
<dt><a href="#module_utils/reports">utils/reports</a> ⇒ <code>Object</code></dt>
<dd><p>Build report utilities</p>
</dd>
<dt><a href="#module_tasks/autoScaffoldTasks">tasks/autoScaffoldTasks</a></dt>
<dd><p>A module to add gulp tasks which automatically update one or more packages.</p>
</dd>
<dt><a href="#module_tasks/autoUpdateDependenciesTasks">tasks/autoUpdateDependenciesTasks</a></dt>
<dd><p>A module to add gulp tasks which automatically update one or more packages.</p>
</dd>
<dt><a href="#module_tasks/updateDependenciesTasks">tasks/updateDependenciesTasks</a></dt>
<dd><p>A module to add gulp tasks which automatically update package dependencies.</p>
</dd>
<dt><a href="#module_tasks/codeAnalysisTasks">tasks/codeAnalysisTasks</a></dt>
<dd><p>A module to add gulp tasks which execute static code analysis.</p>
</dd>
<dt><a href="#module_tasks/coverageStatsTasks">tasks/coverageStatsTasks</a></dt>
<dd><p>A module to add a gulp task which calculates coverage stats from the Istanbul reporter json-summary.</p>
</dd>
<dt><a href="#module_tasks/defaultTasks">tasks/defaultTasks</a></dt>
<dd><p>A module to add a gulp task which executes the default task.</p>
</dd>
<dt><a href="#module_tasks/testTasks">tasks/testTasks</a></dt>
<dd><p>A module to add gulp tasks which run test steps.</p>
</dd>
<dt><a href="#module_utils/coverageStats">utils/coverageStats</a> ⇒ <code>Object</code></dt>
<dd><p>Coverage statistic utilities</p>
</dd>
<dt><a href="#module_tasks/allTasks">tasks/allTasks</a></dt>
<dd><p>A module to add a gulp task which executes all build tasks.</p>
</dd>
<dt><a href="#module_tasks/codeAnalysisTasks">tasks/codeAnalysisTasks</a></dt>
<dd><p>A module to add gulp tasks which execute static code analysis.</p>
</dd>
<dt><a href="#module_tasks/coverageStatsTasks">tasks/coverageStatsTasks</a></dt>
<dd><p>A module to add a gulp task which calculates coverage stats from the Istanbul reporter json-summary.</p>
</dd>
<dt><a href="#module_tasks/davidTasks">tasks/davidTasks</a></dt>
<dd><p>A module to add gulp tasks which check/report/update package dependencies.</p>
</dd>
<dt><a href="#module_tasks/defaultTasks">tasks/defaultTasks</a></dt>
<dd><p>A module to add a default gulp task which executes default build tasks.</p>
</dd>
<dt><a href="#module_tasks/docsTasks">tasks/docsTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare readme documentation.</p>
</dd>
<dt><a href="#module_tasks/docsChangelogTasks">tasks/docsChangelogTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare changelog readme documentation.</p>
</dd>
<dt><a href="#module_tasks/docsJsdocsTasks">tasks/docsJsdocsTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare code api readme documentation.</p>
</dd>
<dt><a href="#module_tasks/docsLicenseTasks">tasks/docsLicenseTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare license readme documentation.</p>
</dd>
<dt><a href="#module_tasks/metadataTasks">tasks/metadataTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare the package.json file for build packaging and deployment.</p>
</dd>
<dt><a href="#module_tasks/nullTasks">tasks/nullTasks</a></dt>
<dd><p>A module to add a gulp task which does nothing.</p>
</dd>
<dt><a href="#module_tasks/packageTasks">tasks/packageTasks</a></dt>
<dd><p>A module to add gulp tasks which prepare the build package.</p>
</dd>
<dt><a href="#module_tasks/postBuild">tasks/postBuild</a></dt>
<dd><p>A module to add gulp tasks which runs post build verification tests</p>
</dd>
<dt><a href="#module_tasks/publishTasks">tasks/publishTasks</a></dt>
<dd><p>A module to add a gulp task which executes publish tasks.</p>
</dd>
<dt><a href="#module_tasks/scaffoldTasks">tasks/scaffoldTasks</a></dt>
<dd><p>A module to add gulp tasks which currently update development dependencies but in future could provide further
scaffolding.</p>
</dd>
<dt><a href="#module_tasks/stepSyncTasks">tasks/stepSyncTasks</a></dt>
<dd><p>A module to add gulp tasks which synchronise test steps from feature files with JIRA.</p>
</dd>
<dt><a href="#module_tasks/stepsTasks">tasks/stepsTasks</a></dt>
<dd><p>A module to add a gulp tasks which creates missing test steps in new or existing step libraries.</p>
</dd>
<dt><a href="#module_tasks/webpackTasks">tasks/webpackTasks</a></dt>
<dd><p>A module to add gulp tasks for the webpack module bundler.</p>
</dd>
</dl>
<a name="module_reports/autoUpdateMocha"></a>
### reports/autoUpdateMocha ⇒ <code>Object</code>
Auto update mocha report

**Returns**: <code>Object</code> - Auto update mocha report functions  

* [reports/autoUpdateMocha](#module_reports/autoUpdateMocha) ⇒ <code>Object</code>
  * [`.prepare`](#module_reports/autoUpdateMocha.prepare) ⇒ <code>Object</code>
  * [`.writeToFileSync`](#module_reports/autoUpdateMocha.writeToFileSync)
  * [`.write(workflowHistory, rpt)`](#module_reports/autoUpdateMocha.write)


-

<a name="module_reports/autoUpdateMocha.prepare"></a>
#### `reports/autoUpdateMocha.prepare` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static property of <code>[reports/autoUpdateMocha](#module_reports/autoUpdateMocha)</code>  
**Returns**: <code>Object</code> - the report header  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| [start] | <code>String</code> | <code>new global.Date()</code> | start time and date |
| [end] | <code>String</code> | <code>new global.Date()</code> | end time and date (will be overwritten by write report function) |


-

<a name="module_reports/autoUpdateMocha.writeToFileSync"></a>
#### `reports/autoUpdateMocha.writeToFileSync`
Synchronously write a test report to a file location

**Kind**: static property of <code>[reports/autoUpdateMocha](#module_reports/autoUpdateMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| reportPath | <code>String</code> | the file path including filename |
| report | <code>Object</code> | the report object containing report header and results |


-

<a name="module_reports/autoUpdateMocha.write"></a>
#### `reports/autoUpdateMocha.write(workflowHistory, rpt)`
Write a new test

**Kind**: static method of <code>[reports/autoUpdateMocha](#module_reports/autoUpdateMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| workflowHistory | <code>Object</code> | the workflow history returned from an auto update |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/baseCucumber"></a>
### reports/baseCucumber ⇒ <code>Object</code>
Base methods for cucumber reports

**Returns**: <code>Object</code> - Base cucumber report functions  

* [reports/baseCucumber](#module_reports/baseCucumber) ⇒ <code>Object</code>
  * [`.prepare(type, name, description)`](#module_reports/baseCucumber.prepare) ⇒ <code>Object</code>
  * [`.write(suite, pass, message, rpt)`](#module_reports/baseCucumber.write)
  * [`.writeToFileSync(reportPath, report)`](#module_reports/baseCucumber.writeToFileSync)


-

<a name="module_reports/baseCucumber.prepare"></a>
#### `reports/baseCucumber.prepare(type, name, description)` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static method of <code>[reports/baseCucumber](#module_reports/baseCucumber)</code>  
**Returns**: <code>Object</code> - the report header  

| Param | Type | Description |
| --- | --- | --- |
| type | <code>String</code> | the test report type (e.g. feature, bug) |
| name | <code>String</code> | the test report name |
| description | <code>String</code> | the test report description |


-

<a name="module_reports/baseCucumber.write"></a>
#### `reports/baseCucumber.write(suite, pass, message, rpt)`
Write a new test

**Kind**: static method of <code>[reports/baseCucumber](#module_reports/baseCucumber)</code>  

| Param | Type | Description |
| --- | --- | --- |
| suite | <code>String</code> | the test suite name |
| pass | <code>Boolean</code> | has the test passed? |
| message | <code>String</code> | the test description message |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/baseCucumber.writeToFileSync"></a>
#### `reports/baseCucumber.writeToFileSync(reportPath, report)`
Synchronously write a test report to a file location

**Kind**: static method of <code>[reports/baseCucumber](#module_reports/baseCucumber)</code>  

| Param | Type | Description |
| --- | --- | --- |
| reportPath | <code>String</code> | the file path including filename |
| report | <code>Object</code> | the report object containing report header and results |


-

<a name="module_reports/baseMocha"></a>
### reports/baseMocha ⇒ <code>Object</code>
Base methods for mocha reports

**Returns**: <code>Object</code> - Base mocha report functions  

* [reports/baseMocha](#module_reports/baseMocha) ⇒ <code>Object</code>
  * [`.prepare([start], [end])`](#module_reports/baseMocha.prepare) ⇒ <code>Object</code>
  * [`.write(suite, pass, message, rpt)`](#module_reports/baseMocha.write)
  * [`.writeToFileSync(reportPath, report)`](#module_reports/baseMocha.writeToFileSync)


-

<a name="module_reports/baseMocha.prepare"></a>
#### `reports/baseMocha.prepare([start], [end])` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static method of <code>[reports/baseMocha](#module_reports/baseMocha)</code>  
**Returns**: <code>Object</code> - the report header  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| [start] | <code>String</code> | <code>new global.Date()</code> | start time and date |
| [end] | <code>String</code> | <code>new global.Date()</code> | end time and date (will be overwritten by write report function) |


-

<a name="module_reports/baseMocha.write"></a>
#### `reports/baseMocha.write(suite, pass, message, rpt)`
Write a new test

**Kind**: static method of <code>[reports/baseMocha](#module_reports/baseMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| suite | <code>String</code> | the test suite name |
| pass | <code>Boolean</code> | has the test passed? |
| message | <code>String</code> | the test description message |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/baseMocha.writeToFileSync"></a>
#### `reports/baseMocha.writeToFileSync(reportPath, report)`
Synchronously write a test report to a file location

**Kind**: static method of <code>[reports/baseMocha](#module_reports/baseMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| reportPath | <code>String</code> | the file path including filename |
| report | <code>Object</code> | the report object containing report header and results |


-

<a name="module_reports/davidCucumber"></a>
### reports/davidCucumber ⇒ <code>Object</code>
David cucumber report

**Returns**: <code>Object</code> - David cucumber report functions  

* [reports/davidCucumber](#module_reports/davidCucumber) ⇒ <code>Object</code>
  * [`.prepare()`](#module_reports/davidCucumber.prepare) ⇒ <code>Object</code>
  * [`.write(opts, pkgs, pkg, rpt)`](#module_reports/davidCucumber.write)


-

<a name="module_reports/davidCucumber.prepare"></a>
#### `reports/davidCucumber.prepare()` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static method of <code>[reports/davidCucumber](#module_reports/davidCucumber)</code>  
**Returns**: <code>Object</code> - the report header  

-

<a name="module_reports/davidCucumber.write"></a>
#### `reports/davidCucumber.write(opts, pkgs, pkg, rpt)`
Write a new test

**Kind**: static method of <code>[reports/davidCucumber](#module_reports/davidCucumber)</code>  

| Param | Type | Description |
| --- | --- | --- |
| opts | <code>Object</code> | the david options |
| opts.suite | <code>String</code> | the test suite name |
| pkgs | <code>Object</code> | the package dependency analysis returned by david |
| pkg | <code>String</code> | the package to write test for |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/davidMocha"></a>
### reports/davidMocha ⇒ <code>Object</code>
David mocha report

**Returns**: <code>Object</code> - David mocha report functions  

* [reports/davidMocha](#module_reports/davidMocha) ⇒ <code>Object</code>
  * [`.prepare`](#module_reports/davidMocha.prepare) ⇒ <code>Object</code>
  * [`.writeToFileSync`](#module_reports/davidMocha.writeToFileSync)
  * [`.write(opts, pkgs, pkg, rpt)`](#module_reports/davidMocha.write)


-

<a name="module_reports/davidMocha.prepare"></a>
#### `reports/davidMocha.prepare` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static property of <code>[reports/davidMocha](#module_reports/davidMocha)</code>  
**Returns**: <code>Object</code> - the report header  

| Param | Type | Description |
| --- | --- | --- |
| start | <code>String</code> | start time and date |
| end | <code>String</code> | end time and date (will be overwritten by write report function) |


-

<a name="module_reports/davidMocha.writeToFileSync"></a>
#### `reports/davidMocha.writeToFileSync`
Synchronously write a test report to a file location

**Kind**: static property of <code>[reports/davidMocha](#module_reports/davidMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| reportPath | <code>String</code> | the file path including filename |
| report | <code>Object</code> | the report object containing report header and results |


-

<a name="module_reports/davidMocha.write"></a>
#### `reports/davidMocha.write(opts, pkgs, pkg, rpt)`
Write a new test

**Kind**: static method of <code>[reports/davidMocha](#module_reports/davidMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| opts | <code>Object</code> | the david options |
| opts.suite | <code>String</code> | the test suite name |
| pkgs | <code>Object</code> | the package dependency analysis returned by david |
| pkg | <code>String</code> | the package to write test for |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/eslintBunyan"></a>
### reports/eslintBunyan ⇒ <code>String</code>
Eslint bunyan report logger.

**Returns**: <code>String</code> - empty string  

| Param | Type | Description |
| --- | --- | --- |
| results | <code>Array</code> | eslint results returned through gulp-eslint |


-

<a name="module_reports/eslintCucumber"></a>
### reports/eslintCucumber ⇒ <code>Object</code>
Eslint cucumber report

**Returns**: <code>Object</code> - Eslint cucumber report formatter  

| Param | Type | Description |
| --- | --- | --- |
| results | <code>Array</code> | eslint results returned through gulp-eslint |


-

<a name="module_reports/eslintMocha"></a>
### reports/eslintMocha ⇒ <code>Object</code>
Eslint mocha report

**Returns**: <code>Object</code> - Eslint mocha report formatter  

| Param | Type | Description |
| --- | --- | --- |
| results | <code>Array</code> | eslint results returned through gulp-eslint |


-

<a name="module_reports/genericCucumber"></a>
### reports/genericCucumber ⇒ <code>Object</code>
Generic cucumber report

**Returns**: <code>Object</code> - Generic cucumber report functions  

* [reports/genericCucumber](#module_reports/genericCucumber) ⇒ <code>Object</code>
  * [`.prepare`](#module_reports/genericCucumber.prepare) ⇒ <code>Object</code>
  * [`.write(suiteName, testName, errorMessages, givenDesc, whenDesc, thenDesc, rpt)`](#module_reports/genericCucumber.write)


-

<a name="module_reports/genericCucumber.prepare"></a>
#### `reports/genericCucumber.prepare` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static property of <code>[reports/genericCucumber](#module_reports/genericCucumber)</code>  
**Returns**: <code>Object</code> - the report header  

| Param | Type | Description |
| --- | --- | --- |
| type | <code>String</code> | the test report type (e.g. feature, bug) |
| name | <code>String</code> | the test report name |
| description | <code>String</code> | the test report description |


-

<a name="module_reports/genericCucumber.write"></a>
#### `reports/genericCucumber.write(suiteName, testName, errorMessages, givenDesc, whenDesc, thenDesc, rpt)`
Write a new test

**Kind**: static method of <code>[reports/genericCucumber](#module_reports/genericCucumber)</code>  

| Param | Type | Description |
| --- | --- | --- |
| suiteName | <code>String</code> | the test suite name |
| testName | <code>String</code> | the test name |
| errorMessages | <code>Array</code> | the error messages if the test failed |
| givenDesc | <code>String</code> | the given step part description |
| whenDesc | <code>String</code> | the when step part description |
| thenDesc | <code>String</code> | the then step part description |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/genericMocha"></a>
### reports/genericMocha ⇒ <code>Object</code>
Generic mocha report

**Returns**: <code>Object</code> - Generic mocha report functions  

* [reports/genericMocha](#module_reports/genericMocha) ⇒ <code>Object</code>
  * [`.writeToFileSync`](#module_reports/genericMocha.writeToFileSync)
  * [`.prepare()`](#module_reports/genericMocha.prepare) ⇒ <code>Object</code>
  * [`.write(suiteName, testName, errorMessages, rpt)`](#module_reports/genericMocha.write)


-

<a name="module_reports/genericMocha.writeToFileSync"></a>
#### `reports/genericMocha.writeToFileSync`
Synchronously write a test report to a file location

**Kind**: static property of <code>[reports/genericMocha](#module_reports/genericMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| reportPath | <code>String</code> | the file path including filename |
| report | <code>Object</code> | the report object containing report header and results |


-

<a name="module_reports/genericMocha.prepare"></a>
#### `reports/genericMocha.prepare()` ⇒ <code>Object</code>
Prepare the report header

**Kind**: static method of <code>[reports/genericMocha](#module_reports/genericMocha)</code>  
**Returns**: <code>Object</code> - the report header  

-

<a name="module_reports/genericMocha.write"></a>
#### `reports/genericMocha.write(suiteName, testName, errorMessages, rpt)`
Write a new test

**Kind**: static method of <code>[reports/genericMocha](#module_reports/genericMocha)</code>  

| Param | Type | Description |
| --- | --- | --- |
| suiteName | <code>String</code> | the test suite name |
| testName | <code>String</code> | the test name |
| errorMessages | <code>Array</code> | the error messages if the test failed |
| rpt | <code>Object</code> | the report object to write or add the test result to |


-

<a name="module_reports/postBuildCucumber"></a>
### reports/postBuildCucumber ⇒ <code>Object</code>
Post build cucumber report

**Returns**: <code>Object</code> - Post build cucumber report in JSON  

| Param | Type | Description |
| --- | --- | --- |
| suiteName | <code>String</code> | the test suite name |
| results | <code>Array</code> | post build results returned as an array of object test results. The test result object contains two properties `testName` and `errMessage` e.g. `{testName: "test description", errMessage: "error message"}` |
| [reportPath] | <code>String</code> | the file path including filename |


-

<a name="module_reports/postBuildMocha"></a>
### reports/postBuildMocha ⇒ <code>Object</code>
Post build mocha report

**Returns**: <code>Object</code> - Post build mocha report in JSON  

| Param | Type | Description |
| --- | --- | --- |
| suiteName | <code>String</code> | the test suite name |
| results | <code>Array</code> | post build results returned as an array of object test results. The test result object contains two properties `testName` and `errMessage` e.g. `{testName: "test description", errMessage: "error message"}` |
| [reportPath] | <code>String</code> | the file path including filename |


-

<a name="module_utils/autoUpdate"></a>
### utils/autoUpdate ⇒ <code>Object</code>
Auto update build utilities.

**Returns**: <code>Object</code> - Auto update build utility functions  

| Param | Type | Description |
| --- | --- | --- |
| logger | <code>bunyan</code> | A logger matching the bunyan API |


* [utils/autoUpdate](#module_utils/autoUpdate) ⇒ <code>Object</code>
  * [`.run(opts, gulp, context)`](#module_utils/autoUpdate.run) ⇒ <code>through2</code>
  * [`.autoUpdate(opts, cb)`](#module_utils/autoUpdate.autoUpdate)
  * [`.updateProject(opts, cb)`](#module_utils/autoUpdate.updateProject)
  * [`.updateCheck(opts, cb)`](#module_utils/autoUpdate.updateCheck)
  * [`.updateSource(opts, cb)`](#module_utils/autoUpdate.updateSource)
  * [`.releaseBuild(issue, workflowHistory, opts, cb)`](#module_utils/autoUpdate.releaseBuild)


-

<a name="module_utils/autoUpdate.run"></a>
#### `utils/autoUpdate.run(opts, gulp, context)` ⇒ <code>through2</code>
Run the update task on all projects matching specified criteria and generate output reports.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  
**Returns**: <code>through2</code> - stream  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| opts | <code>Object</code> |  | options |
| [opts.repositoryUrl] | <code>String</code> |  | the url to the repository containing packages for checking and update |
| [opts.BUILD_DIR] | <code>String</code> | <code>&quot;Build&quot;</code> | download the repository to this temporary build directory |
| [opts.category] | <code>String</code> | <code>&quot;&quot;</code> | all projects with this JIRA project category are to be included on the update |
| [opts.include] | <code>Array</code> | <code>[]</code> | array of short repository paths to be included in the update |
| [opts.exclude] | <code>Array</code> | <code>[]</code> | array of short repository paths to be excluded in the update |
| [opts.updateCheck] | <code>Boolean</code> | <code>false</code> | flag whether to run tasks to auto-check whether an update is required |
| [opts.updateCheckTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used to auto-check whether an update is required |
| [opts.updateSource] | <code>Boolean</code> | <code>false</code> | flag whether to run tasks to auto-update the source and then commit |
| [opts.updateSourceTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used to auto-update the source |
| [opts.updateSourceSummary] | <code>String</code> | <code>&quot;&quot;</code> | the summary description for the issue created to track source update progress |
| [opts.updateSourceType] | <code>String</code> | <code>&quot;Non-functional&quot;</code> | the issue type for the issue created to track source update progress |
| [opts.bambooBuildTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used for Bamboo build |
| [opts.bambooPostBuildTask] | <code>String</code> | <code>&quot;post_build&quot;</code> | gulp task used for Bamboo post build |
| gulp | <code>function</code> |  | gulp object used to wrap the `autoUpdate` function in a gulp task |
| context | <code>Object</code> |  | context object passed to gulp tasks used to obtain context.cwd - current working directory |


-

<a name="module_utils/autoUpdate.autoUpdate"></a>
#### `utils/autoUpdate.autoUpdate(opts, cb)`
Execute update task on all projects matching specified criteria.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| opts | <code>Object</code> |  | options |
| [opts.repositoryUrl] | <code>String</code> |  | the url to the repository containing packages for checking and update |
| [opts.BUILD_DIR] | <code>String</code> | <code>&quot;Build&quot;</code> | download the repository to this temporary build directory |
| [opts.category] | <code>String</code> | <code>&quot;&quot;</code> | all projects with this JIRA project category are to be included on the update |
| [opts.include] | <code>Array</code> | <code>[]</code> | array of short repository paths to be included in the update |
| [opts.exclude] | <code>Array</code> | <code>[]</code> | array of short repository paths to be excluded in the update |
| [opts.updateCheck] | <code>Boolean</code> | <code>false</code> | flag whether to run tasks to auto-check whether an update is required |
| [opts.updateCheckTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used to auto-check whether an update is required |
| [opts.updateSource] | <code>Boolean</code> | <code>false</code> | flag whether to run tasks to auto-update the source and then commit |
| [opts.updateSourceTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used to auto-update the source |
| [opts.updateSourceSummary] | <code>String</code> | <code>&quot;&quot;</code> | the summary description for the issue created to track source update progress |
| [opts.updateSourceType] | <code>String</code> | <code>&quot;Non-functional&quot;</code> | the issue type for the issue created to track source update progress |
| [opts.bambooBuildTask] | <code>String</code> | <code>&quot;all&quot;</code> | gulp task used for Bamboo build |
| [opts.bambooPostBuildTask] | <code>String</code> | <code>&quot;post_build&quot;</code> | gulp task used for Bamboo post build |
| cb | <code>function</code> |  | callback function with signature: function(err, data) |

**Example**  
```none
Auto update runs a simple workflow for all projects in JIRA matching a specified criteria.

Projects in JIRA are expected to conform to a naming convention which consists of a git repository namespace and repository name: /

```

-

<a name="module_utils/autoUpdate.updateProject"></a>
#### `utils/autoUpdate.updateProject(opts, cb)`
Execute update task on a project.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  

| Param | Type | Description |
| --- | --- | --- |
| opts | <code>Object</code> | options |
| opts.BUILD_DIR | <code>String</code> | download the repository to this temporary build directory |
| opts.repositoryUrl | <code>String</code> | the url to the repository containing packages for checking and update |
| opts.repositoryPath | <code>String</code> | short repository path of project to be processed |
| opts.project | <code>Object</code> | the JIRA project metadata |
| opts.updateCheck | <code>Boolean</code> | flag whether to run tasks to auto-check whether an update is required |
| opts.updateCheckTask | <code>String</code> | gulp task used to auto-check whether an update is required |
| opts.updateSource | <code>Boolean</code> | flag whether to run tasks to auto-update the source and then commit |
| opts.updateSourceTask | <code>String</code> | gulp task used to auto-update the source |
| opts.releaseVersion | <code>Boolean</code> | flag whether to release the patch version for an update |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/autoUpdate.updateCheck"></a>
#### `utils/autoUpdate.updateCheck(opts, cb)`
Check whether an update is required by cloning the master branch of the project to a directory and
running a provided gulp task. The provided gulp task is expected to generate test reports.
The test reports are checked for any failures. If one or more failures found then this function
will return false in the callback data. Otherwise this function will return true in the callback data.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  

| Param | Type | Description |
| --- | --- | --- |
| opts | <code>Object</code> | options |
| opts.BUILD_DIR | <code>String</code> | download the repository to this temporary build directory |
| opts.repositoryUrl | <code>String</code> | the url to the repository containing packages for checking and update |
| opts.repositoryPath | <code>String</code> | short repository path of project to be cloned to the BUILD_DIR |
| opts.updateCheckTask | <code>String</code> | gulp task used to perform the update check |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/autoUpdate.updateSource"></a>
#### `utils/autoUpdate.updateSource(opts, cb)`
Update the source of a project by cloning the master branch of the project and running the provided
gulp task. A JIRA issue is created to track the progress of the update. The issue is transitioned to
In Progress and QA. Once the update is complete the source is committed back to the repository triggering
a Bamboo build on the master branch.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  

| Param | Type | Description |
| --- | --- | --- |
| opts | <code>Object</code> | options |
| opts.BUILD_DIR | <code>String</code> | download the repository to this temporary build directory |
| opts.repositoryUrl | <code>String</code> | the url to the repository containing packages for checking and update |
| opts.repositoryPath | <code>String</code> | short repository path of project to be processed |
| opts.updateSourceTask | <code>String</code> | gulp task used to auto-update the source |
| opts.updateSourceSummary | <code>String</code> | the summary description for the issue created to track source update progress |
| opts.updateSourceType | <code>String</code> | the issue type for the issue created to track source update progress |
| cb | <code>function</code> | callback function with signature: function(err, issue) |


-

<a name="module_utils/autoUpdate.releaseBuild"></a>
#### `utils/autoUpdate.releaseBuild(issue, workflowHistory, opts, cb)`
Transition issue to "done", release a Jira project version, and if there are no unresolved issues
then trigger a release build.

**Kind**: static method of <code>[utils/autoUpdate](#module_utils/autoUpdate)</code>  

| Param | Type | Description |
| --- | --- | --- |
| issue | <code>Object</code> | JIRA issue object map containing the properties `key` and `version` |
| workflowHistory | <code>Object</code> | workflow history object map which will have following properties updated: workflowHistory.transitionedIssue - true if issues transitioned to Done workflowHistory.releasedVersion - JIRA version object map where property `released` will be true if there are no unresolved issues workflowHistory.bambooBuildResults - Bamboo build results object map |
| opts | <code>Object</code> | options |
| opts.project | <code>String</code> | JIRA project object map containing properties `key` and `name` |
| opts.buildPlanName | <code>String</code> | Bamboo build plan name |
| cb | <code>function</code> | callback function with signature: function(err, result) |


-

<a name="module_utils/bamboo"></a>
### utils/bamboo ⇒ <code>Object</code>
Bamboo build utilities

**Returns**: <code>Object</code> - Bamboo build utility functions  

| Param | Type | Description |
| --- | --- | --- |
| logger | <code>bunyan</code> | A logger matching the bunyan API |


* [utils/bamboo](#module_utils/bamboo) ⇒ <code>Object</code>
  * [`.getBuildPlans(cb)`](#module_utils/bamboo.getBuildPlans)
  * [`.queueBuild(buildPlanKey, variables, cb)`](#module_utils/bamboo.queueBuild)
  * [`.getResults(buildPlanKey, maxResults, cb)`](#module_utils/bamboo.getResults)
  * [`.getResult(buildPlanKey, buildNumber, cb)`](#module_utils/bamboo.getResult)
  * [`.triggerBamboo(buildPlanName, variables, cb)`](#module_utils/bamboo.triggerBamboo)


-

<a name="module_utils/bamboo.getBuildPlans"></a>
#### `utils/bamboo.getBuildPlans(cb)`
Get build plans from Bamboo.

**Kind**: static method of <code>[utils/bamboo](#module_utils/bamboo)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/bamboo.queueBuild"></a>
#### `utils/bamboo.queueBuild(buildPlanKey, variables, cb)`
Queue a build on Bamboo.

**Kind**: static method of <code>[utils/bamboo](#module_utils/bamboo)</code>  

| Param | Type | Description |
| --- | --- | --- |
| buildPlanKey | <code>String</code> | key for the build plan to be built |
| variables | <code>Object</code> | an object map containing variables to be past to the bamboo build |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/bamboo.getResults"></a>
#### `utils/bamboo.getResults(buildPlanKey, maxResults, cb)`
Get the queue results of a build project on Bamboo.

**Kind**: static method of <code>[utils/bamboo](#module_utils/bamboo)</code>  

| Param | Type | Description |
| --- | --- | --- |
| buildPlanKey | <code>String</code> | key for the build plan to be built |
| maxResults | <code>Integer</code> | maximum size for returned list |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/bamboo.getResult"></a>
#### `utils/bamboo.getResult(buildPlanKey, buildNumber, cb)`
Get the result of a build queued on Bamboo.

**Kind**: static method of <code>[utils/bamboo](#module_utils/bamboo)</code>  

| Param | Type | Description |
| --- | --- | --- |
| buildPlanKey | <code>String</code> | key for the build plan to be built |
| buildNumber | <code>String</code> | the build number to get results from |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/bamboo.triggerBamboo"></a>
#### `utils/bamboo.triggerBamboo(buildPlanName, variables, cb)`
Trigger a build on Bamboo and return the result in the callback.

**Kind**: static method of <code>[utils/bamboo](#module_utils/bamboo)</code>  

| Param | Type | Description |
| --- | --- | --- |
| buildPlanName | <code>String</code> | the name of the build plan |
| variables | <code>Object</code> | an object map containing variables to be past to the bamboo build |
| cb | <code>function</code> | callback function with signature: function(err, result) |


-

<a name="module_utils/config"></a>
### utils/config ⇒ <code>Object</code>
Application configuration utility

**Returns**: <code>Object</code> - configuration for application  

| Param | Type | Description |
| --- | --- | --- |
| application | <code>String</code> | application |


-

<a name="module_utils/david"></a>
### utils/david ⇒ <code>Object</code>
David build utilities

**Returns**: <code>Object</code> - David build utility functions  

| Param | Type | Description |
| --- | --- | --- |
| logger | <code>bunyan</code> | A logger matching the bunyan API |


* [utils/david](#module_utils/david) ⇒ <code>Object</code>
  * [`.dependencyReport(manifest, [opts], cb)`](#module_utils/david.dependencyReport)
  * [`.recordDependencies(manifest, [opts], rpt, reportWriter, cb)`](#module_utils/david.recordDependencies)
  * [`.addUpdatedDeps(pathToManifest, [opts], cb)`](#module_utils/david.addUpdatedDeps)


-

<a name="module_utils/david.dependencyReport"></a>
#### `utils/david.dependencyReport(manifest, [opts], cb)`
Generate a dependency report in the mocha output format.

**Kind**: static method of <code>[utils/david](#module_utils/david)</code>  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| manifest | <code>Object</code> |  | Parsed package.json file contents |
| [opts] | <code>Object</code> |  | Options |
| [opts.reportType] | <code>Object</code> | <code>&quot;mocha&quot;</code> | The report output format.  Choose from mocha or cucumber. |
| [opts.reportPath] | <code>Object</code> |  | The output path for the report |
| [opts.stable] | <code>Boolean</code> | <code>false</code> | Consider only stable packages |
| cb | <code>function</code> |  | Function that receives the results |


-

<a name="module_utils/david.recordDependencies"></a>
#### `utils/david.recordDependencies(manifest, [opts], rpt, reportWriter, cb)`
Record the result of dependency checks against all provided dependencies of a particular typein the report object. The david.isUpdated() function.

**Kind**: static method of <code>[utils/david](#module_utils/david)</code>  

| Param | Type | Description |
| --- | --- | --- |
| manifest | <code>Object</code> | Parsed package.json file contents |
| [opts] | <code>Object</code> | Options |
| [opts.suite] | <code>String</code> | the dependency type string description |
| [opts.stable] | <code>Boolean</code> | Consider only stable packages |
| [opts.dev] | <code>Boolean</code> | Consider devDependencies |
| [opts.optional] | <code>Boolean</code> | Consider optionalDependencies |
| [opts.peer] | <code>Boolean</code> | Consider peerDependencies |
| [opts.loose] | <code>Boolean</code> | Use loose option when querying semver |
| [opts.npm] | <code>Object</code> | npm configuration options |
| [opts.warn.E404] | <code>Boolean</code> | Collect 404s but don't abort |
| rpt | <code>Object</code> | report object to record results to |
| reportWriter | <code>Object</code> | Provides write method to write the result of each package dependency check to the rpt object |
| cb | <code>function</code> | Function that receives the results |


-

<a name="module_utils/david.addUpdatedDeps"></a>
#### `utils/david.addUpdatedDeps(pathToManifest, [opts], cb)`
Add updated dependencies to package.json

**Kind**: static method of <code>[utils/david](#module_utils/david)</code>  

| Param | Type | Description |
| --- | --- | --- |
| pathToManifest | <code>Object</code> | path to the package.json file |
| [opts] | <code>Object</code> | Options |
| [opts.stable] | <code>Boolean</code> | Consider only stable packages |
| [opts.dev] | <code>Boolean</code> | Provided dependencies are dev dependencies |
| [opts.optional] | <code>Boolean</code> | Provided dependencies are optional dependencies |
| [opts.peer] | <code>Boolean</code> | Consider peerDependencies |
| [opts.loose] | <code>Boolean</code> | Use loose option when querying semver |
| [opts.npm] | <code>Object</code> | npm configuration options |
| [opts.warn.E404] | <code>Boolean</code> | Collect 404s but don"t abort |
| cb | <code>function</code> | Callback |


-

<a name="module_utils/jira"></a>
### utils/jira ⇒ <code>Object</code>
JIRA build utilities

**Returns**: <code>Object</code> - JIRA build utility functions  

* [utils/jira](#module_utils/jira) ⇒ <code>Object</code>
  * [`.rest(opts, cb)`](#module_utils/jira.rest)
  * [`.getProjects(cb)`](#module_utils/jira.getProjects)
  * [`.createIssue(issue, cb)`](#module_utils/jira.createIssue)
  * [`.deleteIssue(key, cb)`](#module_utils/jira.deleteIssue)
  * [`.createVersion(key, version, cb)`](#module_utils/jira.createVersion)
  * [`.deleteVersion(key, version, cb)`](#module_utils/jira.deleteVersion)
  * [`.releaseVersion(key, version, releaseDate, cb)`](#module_utils/jira.releaseVersion)
  * [`.transitionIssue(key, transition, cb)`](#module_utils/jira.transitionIssue)
  * [`.getNextUnreleasedPatchVersion(key, cb)`](#module_utils/jira.getNextUnreleasedPatchVersion)
  * [`.getChangelog(key, cb)`](#module_utils/jira.getChangelog)
  * [`.prepareChangeLogJSON(data)`](#module_utils/jira.prepareChangeLogJSON) ⇒ <code>Object</code>


-

<a name="module_utils/jira.rest"></a>
#### `utils/jira.rest(opts, cb)`
Send query to rest api.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| opts | <code>Object</code> |  | required options |
| [opts.query] | <code>Object</code> |  | the rest query url |
| [opts.method] | <code>Object</code> | <code>&quot;get&quot;</code> | optional the http method - one of get, post, put, delete |
| [opts.postData] | <code>Object</code> | <code>&quot;&quot;</code> | optional the post data for create or update queries. |
| cb | <code>function</code> |  | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.getProjects"></a>
#### `utils/jira.getProjects(cb)`
Get all JIRA projects.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.createIssue"></a>
#### `utils/jira.createIssue(issue, cb)`
Create a JIRA issue.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| issue | <code>Object</code> | object map with properties required to create the issue |
| issue.key | <code>String</code> | key for the JIRA project to create the issue within |
| issue.summary | <code>String</code> | the summary description for the issue |
| issue.issueType | <code>String</code> | the name of the issue type |
| issue.version | <code>String</code> | the release version for the issue (fixversion) |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.deleteIssue"></a>
#### `utils/jira.deleteIssue(key, cb)`
Delete a JIRA issue.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the key for the JIRA issue to delete |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.createVersion"></a>
#### `utils/jira.createVersion(key, version, cb)`
Create a new version in a JIRA project.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the JIRA project key |
| version | <code>String</code> | the version (semver) |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.deleteVersion"></a>
#### `utils/jira.deleteVersion(key, version, cb)`
Delete an existing version in a JIRA project.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the JIRA project key |
| version | <code>String</code> | the version (semver) |
| cb | <code>function</code> | callback function with signature: function(err) |


-

<a name="module_utils/jira.releaseVersion"></a>
#### `utils/jira.releaseVersion(key, version, releaseDate, cb)`
Release a version in a JIRA project.The version will only be released if there are 0 unresolved issue linked to version.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the JIRA project key |
| version | <code>String</code> | the version (semver) |
| releaseDate | <code>Date</code> | the release date |
| cb | <code>function</code> | callback function with signature: function(err, versionObj) Where versionObj is an object with the following properties: versionObj.id  - the version id versionObj.name  - the version (semver) versionObj.released  - has the version been released versionObj.releaseDate  - the planned (if released=false) or actual (if released=true) release date versionObj.issuesUnresolvedCount  - the number of unresolved issue linked to version |


-

<a name="module_utils/jira.transitionIssue"></a>
#### `utils/jira.transitionIssue(key, transition, cb)`
Transition a JIRA issue.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | key for the JIRA issue to transition |
| transition | <code>String</code> | the transition id |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.getNextUnreleasedPatchVersion"></a>
#### `utils/jira.getNextUnreleasedPatchVersion(key, cb)`
Get the next unresolved and unreleased minor version for the project (based on semver).If there is an existing unresolved and unreleased major or minor version then the earliest is returned.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the JIRA project key |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.getChangelog"></a>
#### `utils/jira.getChangelog(key, cb)`
Get the changelog for a JIRA project.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | the JIRA project key |
| cb | <code>function</code> | callback function with signature: function(err, data) |


-

<a name="module_utils/jira.prepareChangeLogJSON"></a>
#### `utils/jira.prepareChangeLogJSON(data)` ⇒ <code>Object</code>
Tranform raw changelog data from a JQL query into a JSON object.

**Kind**: static method of <code>[utils/jira](#module_utils/jira)</code>  
**Returns**: <code>Object</code> - changelog JSON object  

| Param | Type | Description |
| --- | --- | --- |
| data | <code>Object</code> | raw changelog data |

**Example**  
Raw changelog data:
     ```
     {
     "releases": [
         {
           "version": {
             "self": "https://jira.cellarise.com/rest/api/2/version/10516",
             "id": "10516",
             "name": "0.1.4",
             "archived": false,
             "released": true,
             "releaseDate": "2014-08-28"
           },
           "issues": []
         }
     }
     ```

-

<a name="module_utils/postBuild"></a>
### utils/postBuild ⇒ <code>Object</code>
Post build utilities

**Returns**: <code>Object</code> - Post build utility functions  

| Param | Type | Description |
| --- | --- | --- |
| logger | <code>bunyan</code> | A logger matching the bunyan API |


* [utils/postBuild](#module_utils/postBuild) ⇒ <code>Object</code>
  * [`.pathExists(testPath, cwd, result)`](#module_utils/postBuild.pathExists) ⇒ <code>Boolean</code>
  * [`.pathNotExists(testPath, cwd, result)`](#module_utils/postBuild.pathNotExists) ⇒ <code>Boolean</code>
  * [`.checks(directories, cwd, cb)`](#module_utils/postBuild.checks)


-

<a name="module_utils/postBuild.pathExists"></a>
#### `utils/postBuild.pathExists(testPath, cwd, result)` ⇒ <code>Boolean</code>
Check whether a file or directory path exists and if not record an error message in the provided results object.

**Kind**: static method of <code>[utils/postBuild](#module_utils/postBuild)</code>  
**Returns**: <code>Boolean</code> - true if the file or directory path exists  

| Param | Type | Description |
| --- | --- | --- |
| testPath | <code>String</code> | the path to test from the current working directory |
| cwd | <code>String</code> | the current working directory |
| result | <code>Object</code> | the result object to record an error message if file or directory path does not exist |


-

<a name="module_utils/postBuild.pathNotExists"></a>
#### `utils/postBuild.pathNotExists(testPath, cwd, result)` ⇒ <code>Boolean</code>
Check whether a file or directory path exists and if so record an error message in the provided results object.

**Kind**: static method of <code>[utils/postBuild](#module_utils/postBuild)</code>  
**Returns**: <code>Boolean</code> - true if the file or directory path does not exist  

| Param | Type | Description |
| --- | --- | --- |
| testPath | <code>String</code> | the path to test from the current working directory |
| cwd | <code>String</code> | the current working directory |
| result | <code>Object</code> | the result object to record an error message if file or directory path exists |


-

<a name="module_utils/postBuild.checks"></a>
#### `utils/postBuild.checks(directories, cwd, cb)`
Post build checks including:Check that build directory exists and contains expected contents

**Kind**: static method of <code>[utils/postBuild](#module_utils/postBuild)</code>  

| Param | Type | Description |
| --- | --- | --- |
| directories | <code>Object</code> | the directories listed in package.json |
| cwd | <code>String</code> | the current working directory |
| cb | <code>function</code> | callback |


-

<a name="module_utils/reports"></a>
### utils/reports ⇒ <code>Object</code>
Build report utilities

**Returns**: <code>Object</code> - Build report utility functions  

* [utils/reports](#module_utils/reports) ⇒ <code>Object</code>
  * [`.checkReportsPass(reportDir)`](#module_utils/reports.checkReportsPass) ⇒ <code>Boolean</code>
  * [`.getReportFailures(reportDir)`](#module_utils/reports.getReportFailures) ⇒ <code>Array</code>


-

<a name="module_utils/reports.checkReportsPass"></a>
#### `utils/reports.checkReportsPass(reportDir)` ⇒ <code>Boolean</code>
Check all mocha test reports and return true if all tests passed or false if one or more tests failed.

**Kind**: static method of <code>[utils/reports](#module_utils/reports)</code>  
**Returns**: <code>Boolean</code> - true if all tests passed or false if one or more tests failed  

| Param | Type | Description |
| --- | --- | --- |
| reportDir | <code>String</code> | the directory containing the mocha test reports. |


-

<a name="module_utils/reports.getReportFailures"></a>
#### `utils/reports.getReportFailures(reportDir)` ⇒ <code>Array</code>
Check all mocha test reports and return an array containing all failures.

**Kind**: static method of <code>[utils/reports](#module_utils/reports)</code>  
**Returns**: <code>Array</code> - - return array containing all failures  

| Param | Type | Description |
| --- | --- | --- |
| reportDir | <code>String</code> | the directory containing the mocha test reports. |


-

<a name="module_tasks/autoScaffoldTasks"></a>
### tasks/autoScaffoldTasks
A module to add gulp tasks which automatically update one or more packages.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/autoScaffoldTasks..auto_update_dependencies"></a>
#### `tasks/autoScaffoldTasks~auto_update_dependencies` ⇒ <code>through2</code>
A gulp build task to automatically update one or more packages.

**Kind**: inner property of <code>[tasks/autoScaffoldTasks](#module_tasks/autoScaffoldTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/autoUpdateDependenciesTasks"></a>
### tasks/autoUpdateDependenciesTasks
A module to add gulp tasks which automatically update one or more packages.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/autoUpdateDependenciesTasks..auto_update_dependencies"></a>
#### `tasks/autoUpdateDependenciesTasks~auto_update_dependencies` ⇒ <code>through2</code>
A gulp build task to automatically update one or more packages.

**Kind**: inner property of <code>[tasks/autoUpdateDependenciesTasks](#module_tasks/autoUpdateDependenciesTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/updateDependenciesTasks"></a>
### tasks/updateDependenciesTasks
A module to add gulp tasks which automatically update package dependencies.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


-

<a name="module_tasks/updateDependenciesTasks..update_dependencies"></a>
#### `tasks/updateDependenciesTasks~update_dependencies` : <code>Gulp</code>
A gulp task to automatically update package dependencies.The following tasks are executed in sequence: ["scaffold", "david-update", "david-cpm-update"]The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/updateDependenciesTasks](#module_tasks/updateDependenciesTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/codeAnalysisTasks"></a>
### tasks/codeAnalysisTasks
A module to add gulp tasks which execute static code analysis.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)
  * [`~code_analysis`](#module_tasks/codeAnalysisTasks..code_analysis) ⇒ <code>through2</code>
  * [`~code_analysis`](#module_tasks/codeAnalysisTasks..code_analysis) ⇒ <code>through2</code>


-

<a name="module_tasks/codeAnalysisTasks..code_analysis"></a>
#### `tasks/codeAnalysisTasks~code_analysis` ⇒ <code>through2</code>
A gulp build task to execute static code analysis on the files at `package.json:directories.lib`.
The report results are saved to `package.json:directories.reports`

**Kind**: inner property of <code>[tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/codeAnalysisTasks..code_analysis"></a>
#### `tasks/codeAnalysisTasks~code_analysis` ⇒ <code>through2</code>
A gulp build task to execute static code analysis on the files at `package.json:directories.lib`.The report results are saved to `package.json:directories.reports`

**Kind**: inner property of <code>[tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/coverageStatsTasks"></a>
### tasks/coverageStatsTasks
A module to add a gulp task which calculates coverage stats from the Istanbul reporter json-summary.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>json</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)
  * [`~coverage_stats`](#module_tasks/coverageStatsTasks..coverage_stats) ⇒ <code>through2</code>
  * [`~coverage_stats`](#module_tasks/coverageStatsTasks..coverage_stats) ⇒ <code>through2</code>


-

<a name="module_tasks/coverageStatsTasks..coverage_stats"></a>
#### `tasks/coverageStatsTasks~coverage_stats` ⇒ <code>through2</code>
A gulp build task to calculate coverage stats from the Istanbul reporter json-summary.
Coverage stats are appended to package.json config.coverage.stats property.
The coverage stats include an overall coverage percentage and badge colour.

**Kind**: inner property of <code>[tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/coverageStatsTasks..coverage_stats"></a>
#### `tasks/coverageStatsTasks~coverage_stats` ⇒ <code>through2</code>
A gulp build task to calculate coverage stats from the Istanbul reporter json-summary.
Coverage stats are appended to package.json config.coverage.stats property.
The coverage stats include an overall coverage percentage and badge colour.

**Kind**: inner property of <code>[tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/defaultTasks"></a>
### tasks/defaultTasks
A module to add a gulp task which executes the default task.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


* [tasks/defaultTasks](#module_tasks/defaultTasks)
  * [`~default`](#module_tasks/defaultTasks..default) : <code>Gulp</code>
  * [`~default`](#module_tasks/defaultTasks..default) : <code>Gulp</code>


-

<a name="module_tasks/defaultTasks..default"></a>
#### `tasks/defaultTasks~default` : <code>Gulp</code>
A gulp build task to run the default tasks.
The following tasks are executed in sequence: ["test"]
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/defaultTasks](#module_tasks/defaultTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/defaultTasks..default"></a>
#### `tasks/defaultTasks~default` : <code>Gulp</code>
The default private gulp build task to execute all tasks. The following tasks are executed in sequence:["code_analysis", "step_sync", "test_cover", "coverage_stats", "license", "docs", "metadata", "package"]This default task if present will override the default gulp task.The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/defaultTasks](#module_tasks/defaultTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/testTasks"></a>
### tasks/testTasks
A module to add gulp tasks which run test steps.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/testTasks](#module_tasks/testTasks)
  * [`~instrument`](#module_tasks/testTasks..instrument) ⇒ <code>through2</code>
  * [`~test_cover`](#module_tasks/testTasks..test_cover) ⇒ <code>through2</code>
  * [`~test`](#module_tasks/testTasks..test) ⇒ <code>through2</code>


-

<a name="module_tasks/testTasks..instrument"></a>
#### `tasks/testTasks~instrument` ⇒ <code>through2</code>
A gulp build task to instrument files.
Istanbul will override the node require() function to redirect to the instrumented files.

**Kind**: inner property of <code>[tasks/testTasks](#module_tasks/testTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/testTasks..test_cover"></a>
#### `tasks/testTasks~test_cover` ⇒ <code>through2</code>
A gulp build task to run test steps and calculate test coverage.
Test steps results will be output using mocha-bamboo-reporter-bgo reporter.
This task executes the Instrument task as a prerequisite.

**Kind**: inner property of <code>[tasks/testTasks](#module_tasks/testTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/testTasks..test"></a>
#### `tasks/testTasks~test` ⇒ <code>through2</code>
A gulp build task to run test steps and calculate test coverage.
Test steps results will be output using spec reporter.

**Kind**: inner property of <code>[tasks/testTasks](#module_tasks/testTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_utils/coverageStats"></a>
### utils/coverageStats ⇒ <code>Object</code>
Coverage statistic utilities

**Returns**: <code>Object</code> - coverage statistics utility functions  

| Param | Type | Description |
| --- | --- | --- |
| logger | <code>bunyan</code> | A logger matching the bunyan API |


* [utils/coverageStats](#module_utils/coverageStats) ⇒ <code>Object</code>
  * [`.addStats(collection, pkg)`](#module_utils/coverageStats.addStats)
  * [`.deleteStats(collection)`](#module_utils/coverageStats.deleteStats)
  * [`.badgeColour(collection, stat, watermarks)`](#module_utils/coverageStats.badgeColour)
  * [`.calculateCoverageStats(coverageReport, packageJSON)`](#module_utils/coverageStats.calculateCoverageStats) ⇒ <code>Object</code>


-

<a name="module_utils/coverageStats.addStats"></a>
#### `utils/coverageStats.addStats(collection, pkg)`
Helper function to append statistic properties from the provided collection to the provided package.json

**Kind**: static method of <code>[utils/coverageStats](#module_utils/coverageStats)</code>  

| Param | Type | Description |
| --- | --- | --- |
| collection | <code>Object</code> | a collection of statistic properties |
| pkg | <code>Object</code> | package.json object |


-

<a name="module_utils/coverageStats.deleteStats"></a>
#### `utils/coverageStats.deleteStats(collection)`
Helper function to delete total, covered and skipped statistic properties from a collection

**Kind**: static method of <code>[utils/coverageStats](#module_utils/coverageStats)</code>  

| Param | Type | Description |
| --- | --- | --- |
| collection | <code>Object</code> | a collection of statistic properties |


-

<a name="module_utils/coverageStats.badgeColour"></a>
#### `utils/coverageStats.badgeColour(collection, stat, watermarks)`
Helper function to determine badge colour

**Kind**: static method of <code>[utils/coverageStats](#module_utils/coverageStats)</code>  

| Param | Type | Description |
| --- | --- | --- |
| collection | <code>Object</code> | a collection of statistic properties |
| stat | <code>Object</code> | a statistic from the collection to calculate the badge for |
| watermarks | <code>Object</code> | the high and low watermarks for each statistic in collection |


-

<a name="module_utils/coverageStats.calculateCoverageStats"></a>
#### `utils/coverageStats.calculateCoverageStats(coverageReport, packageJSON)` ⇒ <code>Object</code>
Calculate coverage stats from an istanbul coverage.json reportand append to provided package.json config.coverage.stats property.The coverage stats include an overall coverage percentage and badge colour.

**Kind**: static method of <code>[utils/coverageStats](#module_utils/coverageStats)</code>  
**Returns**: <code>Object</code> - updated package.json object  

| Param | Type | Description |
| --- | --- | --- |
| coverageReport | <code>Object</code> | the istanbul generated coverage.json report object |
| packageJSON | <code>Object</code> | the package.json object |


-

<a name="module_tasks/allTasks"></a>
### tasks/allTasks
A module to add a gulp task which executes all build tasks.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


* [tasks/allTasks](#module_tasks/allTasks)
  * [`~all`](#module_tasks/allTasks..all) : <code>Gulp</code>
  * [`~all_product`](#module_tasks/allTasks..all_product) : <code>Gulp</code>
  * [`~all_product`](#module_tasks/allTasks..all_product) : <code>Gulp</code>


-

<a name="module_tasks/allTasks..all"></a>
#### `tasks/allTasks~all` : <code>Gulp</code>
A gulp build task to run all build tasks for a module.
The following tasks are executed in sequence:
['code_analysis', 'test_cover', 'coverage_stats', 'license', 'docs', 'metadata', 'package']
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/allTasks](#module_tasks/allTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/allTasks..all_product"></a>
#### `tasks/allTasks~all_product` : <code>Gulp</code>
A gulp build task to run all build tasks for a product.
The following tasks are executed in sequence:
['code_analysis', 'test_cover', 'coverage_stats', 'license', 'docs', 'metadata', 'webpack', 'package']
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/allTasks](#module_tasks/allTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/allTasks..all_product"></a>
#### `tasks/allTasks~all_product` : <code>Gulp</code>
A gulp build task to run webpack and test tasks
The following tasks are executed in sequence:
[ 'webpack', 'test']
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/allTasks](#module_tasks/allTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/codeAnalysisTasks"></a>
### tasks/codeAnalysisTasks
A module to add gulp tasks which execute static code analysis.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)
  * [`~code_analysis`](#module_tasks/codeAnalysisTasks..code_analysis) ⇒ <code>through2</code>
  * [`~code_analysis`](#module_tasks/codeAnalysisTasks..code_analysis) ⇒ <code>through2</code>


-

<a name="module_tasks/codeAnalysisTasks..code_analysis"></a>
#### `tasks/codeAnalysisTasks~code_analysis` ⇒ <code>through2</code>
A gulp build task to execute static code analysis on the files at `package.json:directories.lib`.
The report results are saved to `package.json:directories.reports`

**Kind**: inner property of <code>[tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/codeAnalysisTasks..code_analysis"></a>
#### `tasks/codeAnalysisTasks~code_analysis` ⇒ <code>through2</code>
A gulp build task to execute static code analysis on the files at `package.json:directories.lib`.The report results are saved to `package.json:directories.reports`

**Kind**: inner property of <code>[tasks/codeAnalysisTasks](#module_tasks/codeAnalysisTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/coverageStatsTasks"></a>
### tasks/coverageStatsTasks
A module to add a gulp task which calculates coverage stats from the Istanbul reporter json-summary.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>json</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)
  * [`~coverage_stats`](#module_tasks/coverageStatsTasks..coverage_stats) ⇒ <code>through2</code>
  * [`~coverage_stats`](#module_tasks/coverageStatsTasks..coverage_stats) ⇒ <code>through2</code>


-

<a name="module_tasks/coverageStatsTasks..coverage_stats"></a>
#### `tasks/coverageStatsTasks~coverage_stats` ⇒ <code>through2</code>
A gulp build task to calculate coverage stats from the Istanbul reporter json-summary.
Coverage stats are appended to package.json config.coverage.stats property.
The coverage stats include an overall coverage percentage and badge colour.

**Kind**: inner property of <code>[tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/coverageStatsTasks..coverage_stats"></a>
#### `tasks/coverageStatsTasks~coverage_stats` ⇒ <code>through2</code>
A gulp build task to calculate coverage stats from the Istanbul reporter json-summary.
Coverage stats are appended to package.json config.coverage.stats property.
The coverage stats include an overall coverage percentage and badge colour.

**Kind**: inner property of <code>[tasks/coverageStatsTasks](#module_tasks/coverageStatsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/davidTasks"></a>
### tasks/davidTasks
A module to add gulp tasks which check/report/update package dependencies.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/davidTasks](#module_tasks/davidTasks)
  * [`~david_report`](#module_tasks/davidTasks..david_report) : <code>Gulp</code>
  * [`~david_report_all`](#module_tasks/davidTasks..david_report_all) : <code>Gulp</code>
  * [`~david_update`](#module_tasks/davidTasks..david_update) : <code>Gulp</code>
  * [`~david_update_all`](#module_tasks/davidTasks..david_update_all) : <code>Gulp</code>
  * [`~david_cpm_update`](#module_tasks/davidTasks..david_cpm_update) : <code>Gulp</code>


-

<a name="module_tasks/davidTasks..david_report"></a>
#### `tasks/davidTasks~david_report` : <code>Gulp</code>
A gulp build task to generate a package dependency report for production dependencies onlyand output to the package reports directory.

**Kind**: inner property of <code>[tasks/davidTasks](#module_tasks/davidTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/davidTasks..david_report_all"></a>
#### `tasks/davidTasks~david_report_all` : <code>Gulp</code>
A gulp build task to generate a package dependency report and output to the package reports directory.

**Kind**: inner property of <code>[tasks/davidTasks](#module_tasks/davidTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/davidTasks..david_update"></a>
#### `tasks/davidTasks~david_update` : <code>Gulp</code>
A gulp build task to update package.json dependencies and optional dependencies with the most recent stableversions.

**Kind**: inner property of <code>[tasks/davidTasks](#module_tasks/davidTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/davidTasks..david_update_all"></a>
#### `tasks/davidTasks~david_update_all` : <code>Gulp</code>
A gulp build task to update package.json dependencies with the most recent stable versions.

**Kind**: inner property of <code>[tasks/davidTasks](#module_tasks/davidTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/davidTasks..david_cpm_update"></a>
#### `tasks/davidTasks~david_cpm_update` : <code>Gulp</code>
A gulp build task to run cpm update to update the package NPM dependencies to the most recent stable versions.

**Kind**: inner property of <code>[tasks/davidTasks](#module_tasks/davidTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/defaultTasks"></a>
### tasks/defaultTasks
A module to add a default gulp task which executes default build tasks.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


* [tasks/defaultTasks](#module_tasks/defaultTasks)
  * [`~default`](#module_tasks/defaultTasks..default) : <code>Gulp</code>
  * [`~default`](#module_tasks/defaultTasks..default) : <code>Gulp</code>


-

<a name="module_tasks/defaultTasks..default"></a>
#### `tasks/defaultTasks~default` : <code>Gulp</code>
A gulp build task to run the default tasks.
The following tasks are executed in sequence: ["test"]
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/defaultTasks](#module_tasks/defaultTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/defaultTasks..default"></a>
#### `tasks/defaultTasks~default` : <code>Gulp</code>
The default private gulp build task to execute all tasks. The following tasks are executed in sequence:["code_analysis", "step_sync", "test_cover", "coverage_stats", "license", "docs", "metadata", "package"]This default task if present will override the default gulp task.The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/defaultTasks](#module_tasks/defaultTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/docsTasks"></a>
### tasks/docsTasks
A module to add gulp tasks which prepare readme documentation.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/docsTasks..docs"></a>
#### `tasks/docsTasks~docs` ⇒ <code>through2</code>
A gulp build task to compile and render the `tasks/templates/readme.dust` document template.The document template readme.dust references four other templates:1) readme-api.md (this file is produced by the `docs_jsdocs` gulp task)2) readme-license.md (this file is produced by the `docs_license` gulp task)3) readme-usage.md (this file is updated manually with installation and usage information)4) readme-changelog.md (this file is produced by the `docs_changelog` gulp task)The result is saved to `README.md`.

**Kind**: inner property of <code>[tasks/docsTasks](#module_tasks/docsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/docsChangelogTasks"></a>
### tasks/docsChangelogTasks
A module to add gulp tasks which prepare changelog readme documentation.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/docsChangelogTasks..docs_changelog"></a>
#### `tasks/docsChangelogTasks~docs_changelog` ⇒ <code>through2</code>
A gulp build task to compile and render the package changelog.The changelog data is automatically sourced from Jira if the oauth config.json file exists andpackage.json file contains property `config.projectCode`.The result is saved to `doc/readme-changelog.md`.

**Kind**: inner property of <code>[tasks/docsChangelogTasks](#module_tasks/docsChangelogTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/docsJsdocsTasks"></a>
### tasks/docsJsdocsTasks
A module to add gulp tasks which prepare code api readme documentation.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/docsJsdocsTasks..docs_jsdocs"></a>
#### `tasks/docsJsdocsTasks~docs_jsdocs` ⇒ <code>through2</code>
A gulp build task to generate JSDoc documentation.The result is saved to `doc/readme-api.md`.

**Kind**: inner property of <code>[tasks/docsJsdocsTasks](#module_tasks/docsJsdocsTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/docsLicenseTasks"></a>
### tasks/docsLicenseTasks
A module to add gulp tasks which prepare license readme documentation.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/docsLicenseTasks..docs_license"></a>
#### `tasks/docsLicenseTasks~docs_license` ⇒ <code>through2</code>
A gulp build task to generate license documentation from all dependent packages.The license data is automatically sourced from the node_modules folder using `npm-license`.The result is saved to `doc/readme-license.md`.

**Kind**: inner property of <code>[tasks/docsLicenseTasks](#module_tasks/docsLicenseTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/metadataTasks"></a>
### tasks/metadataTasks
A module to add gulp tasks which prepare the package.json file for build packaging and deployment.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


-

<a name="module_tasks/metadataTasks..metadata"></a>
#### `tasks/metadataTasks~metadata` ⇒ <code>through2</code>
A gulp build task to prepare the package.json file for build packaging and deployment.
Atlassian Bamboo variables are expected and used to populate the package.json version and config properties.
Also, all optionalDependencies are removed.

**Kind**: inner property of <code>[tasks/metadataTasks](#module_tasks/metadataTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/nullTasks"></a>
### tasks/nullTasks
A module to add a gulp task which does nothing.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


-

<a name="module_tasks/nullTasks..null"></a>
#### `tasks/nullTasks~null` : <code>Gulp</code>
A gulp build task that does nothing. Used for tests.

**Kind**: inner property of <code>[tasks/nullTasks](#module_tasks/nullTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/packageTasks"></a>
### tasks/packageTasks
A module to add gulp tasks which prepare the build package.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/packageTasks..package"></a>
#### `tasks/packageTasks~package` ⇒ <code>through2</code>
A gulp build task to copy files to the `package.json:directories.build` directory.The existing build, tasks and reports directories are ignored.

**Kind**: inner property of <code>[tasks/packageTasks](#module_tasks/packageTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/postBuild"></a>
### tasks/postBuild
A module to add gulp tasks which runs post build verification tests


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/publishTasks"></a>
### tasks/publishTasks
A module to add a gulp task which executes publish tasks.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |


-

<a name="module_tasks/publishTasks..all"></a>
#### `tasks/publishTasks~all` : <code>Gulp</code>
A gulp build task to run all build tasks for a module.
The following tasks are executed in sequence:
['code_analysis', 'test_cover', 'coverage_stats', 'license', 'docs', 'metadata', 'package']
The sequence works by piping each task to the next.

**Kind**: inner property of <code>[tasks/publishTasks](#module_tasks/publishTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/scaffoldTasks"></a>
### tasks/scaffoldTasks
A module to add gulp tasks which currently update development dependencies but in future could provide further
scaffolding.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/scaffoldTasks](#module_tasks/scaffoldTasks)
  * [`~scaffold`](#module_tasks/scaffoldTasks..scaffold) ⇒ <code>through2</code>
  * [`~rebuild_scaffold`](#module_tasks/scaffoldTasks..rebuild_scaffold) : <code>Gulp</code>


-

<a name="module_tasks/scaffoldTasks..scaffold"></a>
#### `tasks/scaffoldTasks~scaffold` ⇒ <code>through2</code>
A gulp build task to scaffold an existing package.

**Kind**: inner property of <code>[tasks/scaffoldTasks](#module_tasks/scaffoldTasks)</code>  
**Returns**: <code>through2</code> - stream  

| Param | Type | Description |
| --- | --- | --- |
| done | <code>function</code> | callback |


-

<a name="module_tasks/scaffoldTasks..rebuild_scaffold"></a>
#### `tasks/scaffoldTasks~rebuild_scaffold` : <code>Gulp</code>
A gulp build task to rebuild the source scaffold files for yeoman generators by copying default files from this
package.

**Kind**: inner property of <code>[tasks/scaffoldTasks](#module_tasks/scaffoldTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| done | <code>function</code> | callback |


-

<a name="module_tasks/stepSyncTasks"></a>
### tasks/stepSyncTasks
A module to add gulp tasks which synchronise test steps from feature files with JIRA.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/stepSyncTasks..step_sync"></a>
#### `tasks/stepSyncTasks~step_sync` : <code>Gulp</code>
A gulp build task to download new test features from JIRA and upload changes to existingfeature files back to JIRA.

**Kind**: inner property of <code>[tasks/stepSyncTasks](#module_tasks/stepSyncTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/stepsTasks"></a>
### tasks/stepsTasks
A module to add a gulp tasks which creates missing test steps in new or existing step libraries.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>Bunyan</code> | A logger matching the bunyan API |


-

<a name="module_tasks/stepsTasks..steps"></a>
#### `tasks/stepsTasks~steps` : <code>Gulp</code>
A gulp build task to create missing test steps in new or existing step libraries.

**Kind**: inner property of <code>[tasks/stepsTasks](#module_tasks/stepsTasks)</code>  

| Param | Type | Description |
| --- | --- | --- |
| cb | <code>function</code> | callback |


-

<a name="module_tasks/webpackTasks"></a>
### tasks/webpackTasks
A module to add gulp tasks for the webpack module bundler.


| Param | Type | Description |
| --- | --- | --- |
| gulp | <code>Gulp</code> | The gulp module |
| context | <code>Object</code> | An object containing the following properties: |
| context.cwd | <code>String</code> | The current working directory |
| context.package | <code>Object</code> | The package.json for the module |
| context.argv | <code>Array</code> | The arguments past to the gulp task |
| context.logger | <code>bunyan</code> | A logger matching the bunyan API |


* [tasks/webpackTasks](#module_tasks/webpackTasks)
  * [`~webpack`](#module_tasks/webpackTasks..webpack) ⇒ <code>through2</code>
  * [`~webpack`](#module_tasks/webpackTasks..webpack) ⇒ <code>through2</code>
  * [`~webpack`](#module_tasks/webpackTasks..webpack) ⇒ <code>through2</code>
  * [`~webpackCompileTemplates`](#module_tasks/webpackTasks..webpackCompileTemplates) ⇒ <code>through2</code>
  * [`~webpackDevServer`](#module_tasks/webpackTasks..webpackDevServer) ⇒ <code>through2</code>


-

<a name="module_tasks/webpackTasks..webpack"></a>
#### `tasks/webpackTasks~webpack` ⇒ <code>through2</code>
A gulp build task to run the webpack module bundler for development.

**Kind**: inner property of <code>[tasks/webpackTasks](#module_tasks/webpackTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/webpackTasks..webpack"></a>
#### `tasks/webpackTasks~webpack` ⇒ <code>through2</code>
A gulp build task to run the webpack module bundler using the test config.

**Kind**: inner property of <code>[tasks/webpackTasks](#module_tasks/webpackTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/webpackTasks..webpack"></a>
#### `tasks/webpackTasks~webpack` ⇒ <code>through2</code>
A gulp build task to run the webpack module bundler for production packaging.

**Kind**: inner property of <code>[tasks/webpackTasks](#module_tasks/webpackTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/webpackTasks..webpackCompileTemplates"></a>
#### `tasks/webpackTasks~webpackCompileTemplates` ⇒ <code>through2</code>
A gulp build task to compile dust templates in directories.client.
The dust templates will be provided one of the following package.json as context:
 Build/package.json
 package.json

**Kind**: inner property of <code>[tasks/webpackTasks](#module_tasks/webpackTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

<a name="module_tasks/webpackTasks..webpackDevServer"></a>
#### `tasks/webpackTasks~webpackDevServer` ⇒ <code>through2</code>
A gulp build task to run the webpack dev server

**Kind**: inner property of <code>[tasks/webpackTasks](#module_tasks/webpackTasks)</code>  
**Returns**: <code>through2</code> - stream  

-

*documented by [jsdoc-to-markdown](https://github.com/75lb/jsdoc-to-markdown)*.
# Changelog

<table style="width:100%;border-spacing:0px;border-collapse:collapse;margin:0px;padding:0px;border-width:0px;">
  <tr>
    <th style="width:20px;text-align:center;"></th>
    <th style="width:80px;text-align:center;">Type</th>
    <th style="width:80px;text-align:left;">ID</th>
    <th style="text-align:left;">Summary</th>
  </tr>
    
<tr>
        <td colspan=4><strong>Version: 0.5.0 - released 2015-08-23</strong></td>
      </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-88</td>
            <td><p>Package: Update dependencies</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-87</td>
            <td><p>Package: Update dependencies</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-86</td>
            <td><p>Package: Update development dependencies</p><p></p></td>
          </tr>
        

<tr>
        <td colspan=4><strong>Version: 0.4.0 - released 2015-05-21</strong></td>
      </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-85</td>
            <td><p>Package: replace gulp-istanbul-custom-reporters with gulp-istanbul</p><p>gulp-istanbul@0.9.0 supports customer reporters</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-84</td>
            <td><p>Test task: Add browser test runner and refactor to a test utility script</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-83</td>
            <td><p>Review jsdoc2markdown formatting</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-79</td>
            <td><p>Functional Testing: Add functional test runner</p><p>As a developer
I can run functional tests on multiple browsers automatically using human-readable BDD test scripts
So that I can efficiently perform blackbox and UAT testing and provide UAT documentation as a byproduct</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-82</td>
            <td><p>Docs task: Add additional source directories /server, /common, and /client</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-81</td>
            <td><p>Code-analysis task: Add additional source directories /server, /common, and /client</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-80</td>
            <td><p>Webpack task: Add tasks for dev, test, and production webpack configs</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-78</td>
            <td><p>Package: Update underscore.js</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-77</td>
            <td><p>Package: Update dependencies</p><p>Update glob and set nonegate: false option</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-76</td>
            <td><p>Webpack task: enable dustjs-helpers for webpack precompiled dust templates</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-75</td>
            <td><p>Package: Update eslint dependency</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-64</td>
            <td><p>Package: gulp-eslint v0.2.0 not resolving .eslintrc to nearest parent </p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-74</td>
            <td><p>Webpack task: Add webpack-traceur-loader</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-73</td>
            <td><p>Metadata task: Add all package.config properties to build package.json</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-72</td>
            <td><p>Package: update dependencies</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-71</td>
            <td><p>Webpack task: Add compile templates task for client directory</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-70</td>
            <td><p>Webpack task: Add tasks to bundle modules for browsers</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-69</td>
            <td><p>Post_build task: Remove node_modules check</p><p>Remove node_modules check as products will be built with node_modules.</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-67</td>
            <td><p>Package: Update copyright and license</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-66</td>
            <td><p>Code-analysis task: Remove spaces from ID fields in cucumber report output</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-65</td>
            <td><p>Auto-update task: Add auto-scaffold task</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-63</td>
            <td><p>Package: Update package dependencies</p><p></p></td>
          </tr>
        

<tr>
        <td colspan=4><strong>Version: 0.3.0 - released 2014-11-16</strong></td>
      </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-62</td>
            <td><p>Exec script: Fix maxbuffer error using commitGit function</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-61</td>
            <td><p>Package: Update package dependencies</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-60</td>
            <td><p>Package task: File copy fails to copy all files</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-59</td>
            <td><p>Package task: Add ignore of IDE tool files</p><p>As a developer
I can package my build ignoring my IDE tool files
So that only necessary files are packaged with no unintended files</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-58</td>
            <td><p>Reports: Update cucumber writeToFileSync to append results to an existing file</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-55</td>
            <td><p>Post_build task: Add post build verification test task</p><p>As a build server
I can run post build verification tests
So that I can catch incomplete builds which failed to complete tests and packaging tasks</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-56</td>
            <td><p>Test: Refactor after() for removing test directories to reduce code repetition </p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-54</td>
            <td><p>Exec script: Fix incomplete error logging</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10412&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Minor</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-53</td>
            <td><p>Jsdocs task: Add yeoman default &#39;apps&#39; directory as source folder for scripts</p><p>As a developer
I can generate API for yeoman generators using jsdoc
So that I can efficiently generate documentation for generators</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-52</td>
            <td><p>CoverageStat task: Calculate coverage summary statistics for visual badge</p><p>As a developer
I can generate coverage summary statistics in my package
So that I can communicate overall coverage summary in readme files as a badge</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-51</td>
            <td><p>npm package v2 upgrade</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-3</td>
            <td><p>Code-analysis task: Implement ESLint for custom static code analysis</p><p>As a developer
I can use ESLint for more sophisticated and custom static code analysis
So that where possible to automate I can ensure code quality and enforce coding standards</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10404&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Performance</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-44</td>
            <td><p>Auto-update task: Optimise scheduled auto-update</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-49</td>
            <td><p>Package: migrate from module async to vasync</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-50</td>
            <td><p>Package: migrate from console.log to the bunyan logger</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-48</td>
            <td><p>Coverage-stats task: Fix updated coverage-stats not being written to readme.md</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-46</td>
            <td><p>Scaffold task: Change function to copy standard templates and files into a package</p><p>As a developer
I can automatically copy standard templates and files into a new package or update an existing package
So that I can efficiently maintain consistent with the standard package structure and leverage the features of the package manager</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-47</td>
            <td><p>Step-sync task: Change sync of JIRA issues to linked Feature issue test feature</p><p>As a developer
I can automatically synchronise JIRA issues (Feature issue type) to linked Feature issue test feature (ignoring Deprecated issues)
So that I can efficiently maintain my test features</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-22</td>
            <td><p>Auto-update task: Add a bulk task runner to update one or more projects</p><p>As a developer
The dependencies of all my packages are automatically updated and run through my build server
So that I can keep my packages up to date with bug fixes and patches</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-43</td>
            <td><p>Scripts: Add Git-fetch-reset script</p><p>As a developer
I can execute git fetch --all and git reset --hard in sequence
So that I can force Overwrite of Untracked Working Tree Files</p></td>
          </tr>
        

<tr>
        <td colspan=4><strong>Version: 0.2.0 - released 2014-10-06</strong></td>
      </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10403&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Bug</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-42</td>
            <td><p>Script cgitsh: Fix git command failing to read environment variables</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-41</td>
            <td><p>Migrate from cygwin to msysgit (with ssh-agent working on Bamboo)</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10411&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Feature</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-30</td>
            <td><p>Step-sync task: Add sync of JIRA issues to linked Feature issue test feature</p><p>As a developer
I can automatically synchronise JIRA issues to linked Feature issue test feature
So that I can efficiently maintain my test features</p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-38</td>
            <td><p>Package: Add missing functional tests</p><p></p></td>
          </tr>
        
<tr>
            <td style="width:20px;padding:0;margin:0;text-align:center;"><img src="https://jira.cellarise.com:80/secure/viewavatar?size=xsmall&amp;avatarId=10419&amp;avatarType=issuetype"/></td>
            <td style="width:80px;text-align:left;">Non-functional</td>
            <td style="width:80px;text-align:left;">MDBLDGEN-40</td>
            <td><p>Node: Update node, python and visual studio on CI server</p><p></p></td>
          </tr>
        

</table>

# License

Cellarise License. All rights not explicitly granted in the license are reserved.

Copyright (c) 2015 Cellarise
## Dependencies
[Base64@0.2.1](&quot;https://github.com/davidchambers/Base64.js&quot;) - [&quot;WTFPL&quot;], [abbrev@1.0.7](&quot;git+ssh://git@github.com/isaacs/abbrev-js&quot;) - &quot;ISC&quot;, [accepts@1.2.12](&quot;git+https://github.com/jshttp/accepts&quot;) - &quot;MIT&quot;, [adm-zip@0.4.7](&quot;https://github.com/cthackers/adm-zip&quot;) - [&quot;MIT&quot;], [after@0.8.1](&quot;https://github.com/Raynos/after&quot;) - &quot;MIT*&quot;, [alter@0.2.0](&quot;git+https://github.com/olov/alter&quot;) - &quot;MIT&quot;, [amdefine@1.0.0](&quot;git+https://github.com/jrburke/amdefine&quot;) - &quot;BSD-3-Clause AND MIT&quot;, [ansi-escape-sequences@1.0.2](&quot;https://github.com/75lb/ansi-escape-sequences&quot;) - &quot;MIT&quot;, [ansi-escape-sequences@2.2.1](&quot;git+https://github.com/75lb/ansi-escape-sequences&quot;) - &quot;MIT&quot;, [ansi-green@0.1.1](&quot;git+https://github.com/jonschlinkert/ansi-green&quot;) - &quot;MIT&quot;, [ansi-regex@1.1.1](&quot;git+https://github.com/sindresorhus/ansi-regex&quot;) - &quot;MIT&quot;, [ansi-regex@2.0.0](&quot;git+https://github.com/sindresorhus/ansi-regex&quot;) - &quot;MIT&quot;, [ansi-styles@2.1.0](&quot;git+https://github.com/chalk/ansi-styles&quot;) - &quot;MIT&quot;, [ansi-wrap@0.1.0](&quot;git+https://github.com/jonschlinkert/ansi-wrap&quot;) - &quot;MIT&quot;, [ansi@0.3.0](&quot;https://github.com/TooTallNate/ansi.js&quot;) - &quot;MIT*&quot;, [ansicolors@0.2.1](&quot;https://github.com/thlorenz/ansicolors&quot;) - &quot;MIT&quot;, [ansicolors@0.3.2](&quot;https://github.com/thlorenz/ansicolors&quot;) - &quot;MIT&quot;, [ansistyles@0.1.3](&quot;https://github.com/thlorenz/ansistyles&quot;) - &quot;MIT&quot;, [anymatch@1.3.0](&quot;git+https://github.com/es128/anymatch&quot;) - &quot;ISC&quot;, [archy@1.0.0](&quot;http://github.com/substack/node-archy&quot;) - &quot;MIT&quot;, [are-we-there-yet@1.0.4](&quot;git+https://github.com/iarna/are-we-there-yet&quot;) - &quot;ISC&quot;, [argparse@1.0.2](&quot;https://github.com/nodeca/argparse&quot;) - &quot;MIT&quot;, [arr-diff@1.1.0](&quot;git+https://github.com/jonschlinkert/arr-diff&quot;) - &quot;MIT&quot;, [arr-flatten@1.0.1](&quot;https://github.com/jonschlinkert/arr-flatten&quot;) - &quot;MIT&quot;, [array-back@1.0.2](&quot;git+https://github.com/75lb/array-back&quot;) - &quot;MIT&quot;, [array-differ@1.0.0](&quot;https://github.com/sindresorhus/array-differ&quot;) - &quot;MIT&quot;, [array-flatten@1.1.1](&quot;https://github.com/blakeembrey/array-flatten&quot;) - &quot;MIT&quot;, [array-index@0.1.1](&quot;https://github.com/TooTallNate/array-index&quot;) - &quot;MIT*&quot;, [array-slice@0.2.3](&quot;https://github.com/jonschlinkert/array-slice&quot;) - &quot;MIT&quot;, [array-tools@1.8.6](&quot;git+https://github.com/75lb/array-tools&quot;) - &quot;MIT&quot;, [array-uniq@1.0.2](&quot;git+https://github.com/sindresorhus/array-uniq&quot;) - &quot;MIT&quot;, [array-unique@0.2.1](&quot;https://github.com/jonschlinkert/array-unique&quot;) - &quot;MIT&quot;, [arraybuffer.slice@0.0.6](&quot;git+ssh://git@github.com/rase-/arraybuffer.slice&quot;) - &quot;MIT*&quot;, [arrify@1.0.0](&quot;git+https://github.com/sindresorhus/arrify&quot;) - &quot;MIT&quot;, [asap@1.0.0]([object Object]) - [&quot;MIT&quot;], [asap@2.0.3](&quot;git+https://github.com/kriskowal/asap&quot;) - &quot;MIT&quot;, [asn1@0.1.11](&quot;https://github.com/mcavage/node-asn1&quot;) - &quot;MIT*&quot;, [assert-plus@0.1.5](&quot;git+https://github.com/mcavage/node-assert-plus&quot;) - &quot;MIT*&quot;, [assert@1.3.0](&quot;https://github.com/defunctzombie/commonjs-assert&quot;) - &quot;MIT&quot;, [assertion-error@1.0.1](&quot;git@github.com:chaijs/assertion-error&quot;) - &quot;MIT&quot;, [ast-traverse@0.1.1](&quot;git+https://github.com/olov/ast-traverse&quot;) - &quot;MIT&quot;, [ast-types@0.8.11](&quot;https://github.com/benjamn/ast-types&quot;) - &quot;MIT&quot;, [ast-types@0.8.5](&quot;https://github.com/benjamn/ast-types&quot;) - &quot;MIT&quot;, [async-each@0.1.6](&quot;https://github.com/paulmillr/async-each&quot;) - &quot;MIT&quot;, [async-some@1.0.2](&quot;git+https://github.com/othiym23/async-some&quot;) - &quot;ISC&quot;, [async@0.2.10](&quot;git+https://github.com/caolan/async&quot;) - [&quot;MIT&quot;], [async@0.9.2](&quot;git+https://github.com/caolan/async&quot;) - &quot;MIT&quot;, [async@1.4.0](&quot;git+https://github.com/caolan/async&quot;) - &quot;MIT&quot;, [async@1.4.2](&quot;git+https://github.com/caolan/async&quot;) - &quot;MIT&quot;, [autoprefixer-core@5.2.1](&quot;git+https://github.com/postcss/autoprefixer-core&quot;) - &quot;MIT&quot;, [aws-sign2@0.5.0](&quot;git+https://github.com/mikeal/aws-sign&quot;) - &quot;Apache*&quot;, [babel-core@5.8.22](&quot;git+https://github.com/babel/babel&quot;) - &quot;MIT&quot;, [babel-loader@5.3.2](&quot;git+https://github.com/babel/babel-loader&quot;) - &quot;MIT&quot;, [babel-plugin-constant-folding@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-constant-folding&quot;) - &quot;MIT&quot;, [babel-plugin-dead-code-elimination@1.0.2](&quot;git+https://github.com/babel-plugins/babel-plugin-dead-code-elimination&quot;) - &quot;MIT&quot;, [babel-plugin-eval@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-eval&quot;) - &quot;MIT&quot;, [babel-plugin-inline-environment-variables@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-inline-environment-variables&quot;) - &quot;MIT&quot;, [babel-plugin-jscript@1.0.4](&quot;git+https://github.com/babel-plugins/babel-plugin-jscript&quot;) - &quot;MIT&quot;, [babel-plugin-member-expression-literals@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-member-expression-literals&quot;) - &quot;MIT&quot;, [babel-plugin-property-literals@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-property-literals&quot;) - &quot;MIT&quot;, [babel-plugin-proto-to-assign@1.0.4](&quot;git+https://github.com/babel-plugins/babel-plugin-proto-to-assign&quot;) - &quot;MIT&quot;, [babel-plugin-react-constant-elements@1.0.3](&quot;git+https://github.com/babel-plugins/babel-plugin-react-constant-elements&quot;) - &quot;MIT&quot;, [babel-plugin-react-display-name@1.0.3](&quot;git+https://github.com/babel-plugins/babel-plugin-react-display-name&quot;) - &quot;MIT&quot;, [babel-plugin-remove-console@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-remove-console&quot;) - &quot;MIT&quot;, [babel-plugin-remove-debugger@1.0.1](&quot;git+https://github.com/babel-plugins/babel-plugin-remove-debugger&quot;) - &quot;MIT&quot;, [babel-plugin-runtime@1.0.7](&quot;git+https://github.com/babel-plugins/babel-plugin-runtime&quot;) - &quot;MIT&quot;, [babel-plugin-undeclared-variables-check@1.0.2](&quot;git+https://github.com/babel-plugins/babel-plugin-undeclared-variables-check&quot;) - &quot;MIT&quot;, [babel-plugin-undefined-to-void@1.1.6](&quot;git+https://github.com/babel-plugins/babel-plugin-undefined-to-void&quot;) - &quot;MIT&quot;, [babylon@5.8.22](&quot;git+https://github.com/babel/babel&quot;) - &quot;MIT&quot;, [backo2@1.0.2](&quot;git+https://github.com/mokesmokes/backo&quot;) - &quot;MIT&quot;, [balanced-match@0.1.0](&quot;https://github.com/juliangruber/balanced-match&quot;) - &quot;MIT&quot;, [balanced-match@0.2.0](&quot;https://github.com/juliangruber/balanced-match&quot;) - &quot;MIT&quot;, [base62@0.1.1](&quot;https://github.com/andrew/base62.js&quot;) - &quot;MIT*&quot;, [base64-arraybuffer@0.1.2](&quot;git+https://github.com/niklasvh/base64-arraybuffer&quot;) - [&quot;MIT&quot;], [base64-js@0.0.8](&quot;https://github.com/beatgammit/base64-js&quot;) - &quot;MIT&quot;, [base64-url@1.2.1](&quot;https://github.com/joaquimserafim/base64-url&quot;) - &quot;ISC&quot;, [base64id@0.1.0](&quot;https://github.com/faeldt/base64id&quot;) - , [basic-auth@1.0.3](&quot;git+https://github.com/jshttp/basic-auth&quot;) - &quot;MIT&quot;, [batch@0.5.2](&quot;git+https://github.com/visionmedia/batch&quot;) - [&quot;MIT&quot;], [beeper@1.1.0](&quot;git+https://github.com/sindresorhus/beeper&quot;) - &quot;MIT&quot;, [benchmark@1.0.0](&quot;git+https://github.com/bestiejs/benchmark.js&quot;) - [&quot;MIT&quot;], [better-assert@1.0.2](&quot;git+https://github.com/visionmedia/better-assert&quot;) - &quot;MIT*&quot;, [big.js@3.1.3](&quot;git+https://github.com/MikeMcl/big.js&quot;) - &quot;MIT&quot;, [binary-extensions@1.3.1](&quot;git+https://github.com/sindresorhus/binary-extensions&quot;) - &quot;MIT&quot;, [bl@1.0.0](&quot;git+https://github.com/rvagg/bl&quot;) - &quot;MIT&quot;, [blob@0.0.2](&quot;git+ssh://git@github.com/rase-/blob&quot;) - , [block-stream@0.0.8](&quot;https://github.com/isaacs/block-stream&quot;) - &quot;ISC&quot;, [bluebird@2.9.34](&quot;https://github.com/petkaantonov/bluebird&quot;) - &quot;MIT&quot;, [boom@2.8.0](&quot;https://github.com/hapijs/boom&quot;) - &quot;BSD-3-Clause&quot;, [brace-expansion@1.1.0](&quot;https://github.com/juliangruber/brace-expansion&quot;) - &quot;MIT&quot;, [braces@1.8.1](&quot;git+https://github.com/jonschlinkert/braces&quot;) - &quot;MIT&quot;, [breakable@1.0.0](&quot;git+https://github.com/olov/breakable&quot;) - &quot;MIT&quot;, [browserify-zlib@0.1.4](&quot;https://github.com/devongovett/browserify-zlib&quot;) - &quot;MIT&quot;, [browserslist@0.4.0](&quot;git+https://github.com/ai/browserslist&quot;) - &quot;MIT&quot;, [buffer@3.4.3](&quot;https://github.com/feross/buffer&quot;) - &quot;MIT&quot;, [bufferstreams@1.0.2](&quot;https://github.com/nfroidure/BufferStreams&quot;) - [&quot;MIT&quot;], [builtins@0.0.7](&quot;git+https://github.com/juliangruber/builtins&quot;) - &quot;MIT&quot;, [bunyan-format@0.2.1](&quot;https://github.com/thlorenz/bunyan-format&quot;) - &quot;MIT&quot;, [bunyan@1.4.0](&quot;https://github.com/trentm/node-bunyan&quot;) - &quot;MIT*&quot;, [callsite@1.0.0]([object Object]) - &quot;MIT*&quot;, [camelcase-keys@1.0.0](&quot;git+https://github.com/sindresorhus/camelcase-keys&quot;) - &quot;MIT&quot;, [camelcase@1.2.1](&quot;git+https://github.com/sindresorhus/camelcase&quot;) - &quot;MIT&quot;, [caniuse-db@1.0.30000273](&quot;https://github.com/Fyrd/caniuse&quot;) - &quot;CC-BY&quot;, [caseless@0.11.0](&quot;git+https://github.com/mikeal/caseless&quot;) - &quot;Apache-2.0&quot;, [catharsis@0.8.7](&quot;git+https://github.com/hegemonic/catharsis&quot;) - &quot;MIT&quot;, [cellarise-package-manager@0.0.0](&quot;https://github.com/Cellarise/cellarise-package-manager&quot;) - &quot;Cellarise License&quot;, [chai@3.2.0](&quot;git+https://github.com/chaijs/chai&quot;) - &quot;MIT&quot;, [chalk@1.1.1](&quot;git+https://github.com/chalk/chalk&quot;) - &quot;MIT&quot;, [char-spinner@1.0.1](&quot;https://github.com/isaacs/char-spinner&quot;) - &quot;ISC&quot;, [chmodr@1.0.1](&quot;https://github.com/isaacs/chmodr&quot;) - &quot;ISC&quot;, [chokidar@1.0.5](&quot;git+https://github.com/paulmillr/chokidar&quot;) - &quot;MIT&quot;, [chownr@1.0.1](&quot;https://github.com/isaacs/chownr&quot;) - &quot;ISC&quot;, [cli-color-tty@2.0.0](&quot;git+https://github.com/alanshaw/cli-color-tty&quot;) - &quot;ISC&quot;, [cli-color@1.0.0](&quot;https://github.com/medikoo/cli-color&quot;) - &quot;MIT&quot;, [cli-table@0.3.1](&quot;git+https://github.com/Automattic/cli-table&quot;) - &quot;MIT*&quot;, [cli-width@1.0.1](&quot;git+ssh://git@github.com/knownasilya/cli-width&quot;) - &quot;ISC&quot;, [cli@0.6.6](&quot;git+ssh://git@github.com/chriso/cli&quot;) - [&quot;MIT&quot;], [clone-stats@0.0.1](&quot;https://github.com/hughsk/clone-stats&quot;) - &quot;MIT&quot;, [clone@0.1.19](&quot;https://github.com/pvorb/node-clone&quot;) - &quot;MIT&quot;, [clone@0.2.0](&quot;https://github.com/pvorb/node-clone&quot;) - &quot;MIT&quot;, [clone@1.0.2](&quot;https://github.com/pvorb/node-clone&quot;) - &quot;MIT&quot;, [cmd-shim@2.0.1](&quot;https://github.com/ForbesLindesay/cmd-shim&quot;) - &quot;BSD&quot;, [collect-all@0.1.0](&quot;git+https://github.com/75lb/collect-all&quot;) - &quot;MIT&quot;, [color-convert@0.5.3](&quot;git+ssh://git@github.com/harthur/color-convert&quot;) - &quot;MIT*&quot;, [color-name@1.0.0](&quot;git+ssh://git@github.com/dfcreative/color-name&quot;) - &quot;unlicensed&quot;, [color-string@0.3.0](&quot;git+ssh://git@github.com/harthur/color-string&quot;) - &quot;MIT&quot;, [color@0.10.1](&quot;git+ssh://git@github.com/harthur/color&quot;) - &quot;MIT&quot;, [colormin@1.0.4](&quot;git+https://github.com/ben-eb/colormin&quot;) - &quot;MIT&quot;, [colors@1.0.3](&quot;git+ssh://git@github.com/Marak/colors.js&quot;) - &quot;MIT&quot;, [columnify@1.5.2](&quot;https://github.com/timoxley/columnify&quot;) - &quot;MIT&quot;, [combined-stream@1.0.5](&quot;https://github.com/felixge/node-combined-stream&quot;) - &quot;MIT&quot;, [command-line-args@0.5.9](&quot;git+https://github.com/75lb/command-line-args&quot;) - &quot;MIT&quot;, [commander@0.6.1](&quot;https://github.com/visionmedia/commander.js&quot;) - &quot;MIT*&quot;, [commander@2.3.0](&quot;git+https://github.com/visionmedia/commander.js&quot;) - &quot;MIT*&quot;, [commander@2.5.1](&quot;git+https://github.com/tj/commander.js&quot;) - &quot;MIT&quot;, [commander@2.8.1](&quot;git+https://github.com/tj/commander.js&quot;) - &quot;MIT&quot;, [commoner@0.10.3](&quot;https://github.com/reactjs/commoner&quot;) - &quot;MIT&quot;, [component-bind@1.0.0](&quot;git+https://github.com/component/bind&quot;) - &quot;MIT*&quot;, [component-emitter@1.1.2](&quot;git+https://github.com/component/emitter&quot;) - &quot;MIT*&quot;, [component-inherit@0.0.3](&quot;git+https://github.com/component/inherit&quot;) - &quot;MIT*&quot;, [concat-map@0.0.1](&quot;https://github.com/substack/node-concat-map&quot;) - &quot;MIT&quot;, [concat-stream@1.5.0](&quot;git+ssh://git@github.com/maxogden/concat-stream&quot;) - &quot;MIT&quot;, [concat-with-sourcemaps@1.0.2](&quot;https://github.com/floridoo/concat-with-sourcemaps&quot;) - &quot;ISC&quot;, [config-chain@1.1.9](&quot;https://github.com/dominictarr/config-chain&quot;) - [&quot;MIT&quot;], [connect-history-api-fallback@1.1.0](&quot;git+ssh://git@github.com/bripkens/connect-history-api-fallback&quot;) - &quot;MIT&quot;, [console-browserify@1.1.0](&quot;https://github.com/Raynos/console-browserify&quot;) - [&quot;MIT&quot;], [console-dope@0.3.6](&quot;git+https://github.com/75lb/console-dope&quot;) - , [constants-browserify@0.0.1](&quot;https://github.com/juliangruber/constants-browserify&quot;) - &quot;MIT&quot;, [content-disposition@0.5.0](&quot;git+https://github.com/jshttp/content-disposition&quot;) - &quot;MIT&quot;, [content-type@1.0.1](&quot;git+https://github.com/jshttp/content-type&quot;) - &quot;MIT&quot;, [convert-source-map@1.1.1](&quot;https://github.com/thlorenz/convert-source-map&quot;) - &quot;MIT&quot;, [cookie-parser@1.3.5](&quot;git+https://github.com/expressjs/cookie-parser&quot;) - &quot;MIT&quot;, [cookie-signature@1.0.6](&quot;git+https://github.com/visionmedia/node-cookie-signature&quot;) - &quot;MIT&quot;, [cookie@0.1.3](&quot;git+https://github.com/jshttp/cookie&quot;) - &quot;MIT&quot;, [core-js@1.1.1](&quot;git+https://github.com/zloirock/core-js&quot;) - &quot;MIT&quot;, [core-util-is@1.0.1](&quot;https://github.com/isaacs/core-util-is&quot;) - &quot;MIT&quot;, [crc@3.3.0](&quot;https://github.com/alexgorbatchev/node-crc&quot;) - &quot;MIT&quot;, [cryptiles@2.0.4](&quot;https://github.com/hapijs/cryptiles&quot;) - [&quot;BSD&quot;], [crypto-browserify@3.2.8](&quot;https://github.com/dominictarr/crypto-browserify&quot;) - &quot;MIT&quot;, [css-color-names@0.0.1](&quot;https://github.com/bahamas10/css-color-names&quot;) - &quot;MIT*&quot;, [css-list@0.1.2](&quot;git+https://github.com/TrySound/css-list&quot;) - &quot;MIT&quot;, [css-loader@0.16.0](&quot;git+ssh://git@github.com/webpack/css-loader&quot;) - &quot;MIT&quot;, [css-selector-tokenizer@0.5.4](&quot;git+https://github.com/css-modules/css-selector-tokenizer&quot;) - &quot;MIT&quot;, [cssesc@0.1.0](&quot;git+https://github.com/mathiasbynens/cssesc&quot;) - [&quot;MIT&quot;], [cssnano@2.6.1](&quot;git+https://github.com/ben-eb/cssnano&quot;) - &quot;MIT&quot;, [ctype@0.5.3](&quot;git+https://github.com/rmustacc/node-ctype&quot;) - &quot;MIT*&quot;, [d@0.1.1](&quot;https://github.com/medikoo/d&quot;) - &quot;MIT&quot;, [date-now@0.1.4](&quot;https://github.com/Colingo/date-now&quot;) - [&quot;MIT&quot;], [dateformat@1.0.11](&quot;git+https://github.com/felixge/node-dateformat&quot;) - &quot;MIT&quot;, [david@6.2.0](&quot;https://github.com/alanshaw/david&quot;) - &quot;MIT&quot;, [ddata@0.1.19](&quot;git+https://github.com/jsdoc2md/ddata&quot;) - , [debug@0.7.4](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT*&quot;, [debug@1.0.2](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT*&quot;, [debug@1.0.3](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT*&quot;, [debug@1.0.4](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT*&quot;, [debug@2.0.0](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT*&quot;, [debug@2.1.0](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT&quot;, [debug@2.2.0](&quot;https://github.com/visionmedia/debug&quot;) - &quot;MIT&quot;, [debuglog@1.0.1](&quot;git+https://github.com/sam-github/node-debuglog&quot;) - &quot;MIT&quot;, [decamelize@1.0.0](&quot;git+https://github.com/sindresorhus/decamelize&quot;) - &quot;MIT&quot;, [deep-eql@0.1.3](&quot;git@github.com:chaijs/deep-eql&quot;) - &quot;MIT&quot;, [deep-is@0.1.3](&quot;git+ssh://git@github.com/thlorenz/deep-is&quot;) - &quot;MIT&quot;, [deepmerge@0.2.10](&quot;https://github.com/nrf110/deepmerge&quot;) - &quot;MIT&quot;, [defaults@1.0.2](&quot;https://github.com/tmpvar/defaults&quot;) - &quot;MIT&quot;, [define-properties@1.1.1](&quot;https://github.com/ljharb/define-properties&quot;) - &quot;MIT&quot;, [defined@1.0.0](&quot;https://github.com/substack/defined&quot;) - &quot;MIT&quot;, [defs@1.1.0](&quot;git+https://github.com/olov/defs&quot;) - &quot;MIT&quot;, [delayed-stream@1.0.0](&quot;https://github.com/felixge/node-delayed-stream&quot;) - &quot;MIT&quot;, [delegates@0.1.0](&quot;https://github.com/visionmedia/node-delegates&quot;) - &quot;MIT&quot;, [depd@1.0.1](&quot;git+https://github.com/dougwilson/nodejs-depd&quot;) - &quot;MIT&quot;, [deprecated@0.0.1](&quot;https://github.com/wearefractal/deprecated&quot;) - [&quot;MIT&quot;], [destroy@1.0.3](&quot;git+https://github.com/stream-utils/destroy&quot;) - &quot;MIT&quot;, [detect-indent@2.0.0](&quot;git+https://github.com/sindresorhus/detect-indent&quot;) - &quot;MIT&quot;, [detect-indent@3.0.1](&quot;git+https://github.com/sindresorhus/detect-indent&quot;) - &quot;MIT&quot;, [dezalgo@1.0.3](&quot;git+https://github.com/npm/dezalgo&quot;) - &quot;ISC&quot;, [diff@1.4.0](&quot;https://github.com/kpdecker/jsdiff&quot;) - [&quot;BSD&quot;], [dmd@1.1.1](&quot;git+https://github.com/75lb/dmd&quot;) - &quot;MIT&quot;, [doctrine@0.6.4](&quot;git+ssh://git@github.com/Constellation/doctrine&quot;) - [&quot;BSD&quot;], [dojo@2.0.0-alpha.6](&quot;git+https://github.com/dojo/dojo2&quot;) - &quot;BSD-3-Clause&quot;, [domain-browser@1.1.4](&quot;http://github.com/bevry/domain-browser&quot;) - &quot;MIT&quot;, [dtrace-provider@0.5.0](&quot;git+ssh://git@github.com/chrisa/node-dtrace-provider&quot;) - &quot;BSD*&quot;, [duplexer2@0.0.2](&quot;https://github.com/deoxxa/duplexer2&quot;) - &quot;BSD&quot;, [dustjs-helpers@1.7.3](&quot;git+https://github.com/linkedin/dustjs-helpers&quot;) - &quot;MIT&quot;, [dustjs-linkedin@2.7.2](&quot;git+https://github.com/linkedin/dustjs&quot;) - &quot;MIT&quot;, [editor@1.0.0](&quot;https://github.com/substack/node-editor&quot;) - &quot;MIT&quot;, [ee-first@1.1.1](&quot;git+https://github.com/jonathanong/ee-first&quot;) - &quot;MIT&quot;, [end-of-stream@0.1.5](&quot;https://github.com/mafintosh/end-of-stream&quot;) - &quot;MIT&quot;, [engine.io-client@1.5.2](&quot;git+https://github.com/Automattic/engine.io-client&quot;) - &quot;MIT*&quot;, [engine.io-parser@1.2.1](&quot;git+ssh://git@github.com/LearnBoost/engine.io-parser&quot;) - &quot;MIT*&quot;, [engine.io@1.5.2](&quot;git+ssh://git@github.com/Automattic/engine.io&quot;) - &quot;MIT*&quot;, [enhanced-resolve@0.9.0](&quot;https://github.com/webpack/enhanced-resolve&quot;) - [&quot;MIT&quot;], [envify@3.4.0](&quot;https://github.com/hughsk/envify&quot;) - &quot;MIT&quot;, [errno@0.1.4](&quot;https://github.com/rvagg/node-errno&quot;) - &quot;MIT&quot;, [errorhandler@1.4.2](&quot;git+https://github.com/expressjs/errorhandler&quot;) - &quot;MIT&quot;, [es5-ext@0.10.7](&quot;https://github.com/medikoo/es5-ext&quot;) - &quot;MIT&quot;, [es6-iterator@0.1.3](&quot;https://github.com/medikoo/es6-iterator&quot;) - &quot;MIT&quot;, [es6-map@0.1.1](&quot;https://github.com/medikoo/es6-map&quot;) - &quot;MIT&quot;, [es6-promise@2.3.0](&quot;https://github.com/jakearchibald/ES6-Promises&quot;) - &quot;MIT&quot;, [es6-set@0.1.1](&quot;https://github.com/medikoo/es6-set&quot;) - &quot;MIT&quot;, [es6-symbol@0.1.1](&quot;https://github.com/medikoo/es6-symbol&quot;) - &quot;MIT&quot;, [es6-symbol@2.0.1](&quot;https://github.com/medikoo/es6-symbol&quot;) - &quot;MIT&quot;, [es6-weak-map@0.1.4](&quot;https://github.com/medikoo/es6-weak-map&quot;) - &quot;MIT&quot;, [escape-html@1.0.2](&quot;git+https://github.com/component/escape-html&quot;) - &quot;MIT&quot;, [escape-string-regexp@1.0.2](&quot;git+https://github.com/sindresorhus/escape-string-regexp&quot;) - &quot;MIT&quot;, [escape-string-regexp@1.0.3](&quot;git+https://github.com/sindresorhus/escape-string-regexp&quot;) - &quot;MIT&quot;, [escodegen@1.6.1](&quot;git+ssh://git@github.com/estools/escodegen&quot;) - [&quot;BSD&quot;], [escope@3.2.0](&quot;git+https://github.com/estools/escope&quot;) - &quot;BSD-2-Clause&quot;, [eslint-plugin-react@3.2.3](&quot;git+https://github.com/yannickcr/eslint-plugin-react&quot;) - &quot;MIT&quot;, [eslint@1.2.1](&quot;git+https://github.com/eslint/eslint&quot;) - &quot;MIT&quot;, [espree@2.2.4](&quot;git+ssh://git@github.com/eslint/espree&quot;) - [&quot;BSD&quot;], [esprima-fb@13001.1001.0-dev-harmony-fb](&quot;git+ssh://git@github.com/facebook/esprima&quot;) - [&quot;BSD&quot;], [esprima-fb@15001.1.0-dev-harmony-fb](&quot;git+ssh://git@github.com/facebook/esprima&quot;) - [&quot;BSD&quot;], [esprima-fb@15001.1001.0-dev-harmony-fb](&quot;http://github.com/facebook/esprima&quot;) - [&quot;BSD&quot;], [esprima-fb@8001.1001.0-dev-harmony-fb](&quot;git+ssh://git@github.com/facebook/esprima&quot;) - [&quot;BSD&quot;], [esprima@1.2.5](&quot;git+ssh://git@github.com/ariya/esprima&quot;) - [&quot;BSD&quot;], [esprima@2.2.0](&quot;git+https://github.com/jquery/esprima&quot;) - [&quot;BSD&quot;], [esprima@2.5.0](&quot;git+https://github.com/jquery/esprima&quot;) - &quot;BSD-2-Clause&quot;, [esrecurse@3.1.1](&quot;git+ssh://git@github.com/estools/esrecurse&quot;) - [&quot;BSD&quot;], [estraverse-fb@1.3.1](&quot;git+https://github.com/RReverser/estraverse-fb&quot;) - &quot;MIT&quot;, [estraverse@1.9.3](&quot;git+ssh://git@github.com/estools/estraverse&quot;) - [&quot;BSD&quot;], [estraverse@3.1.0](&quot;git+ssh://git@github.com/estools/estraverse&quot;) - [&quot;BSD&quot;], [estraverse@4.1.0](&quot;git+ssh://git@github.com/estools/estraverse&quot;) - [&quot;BSD&quot;], [esutils@1.1.6](&quot;git+ssh://git@github.com/Constellation/esutils&quot;) - [&quot;BSD&quot;], [esutils@2.0.2](&quot;git+ssh://git@github.com/estools/esutils&quot;) - [&quot;BSD&quot;], [etag@1.7.0](&quot;git+https://github.com/jshttp/etag&quot;) - &quot;MIT&quot;, [event-emitter@0.3.3](&quot;https://github.com/medikoo/event-emitter&quot;) - &quot;MIT&quot;, [eventemitter3@1.1.1](&quot;https://github.com/primus/eventemitter3&quot;) - &quot;MIT&quot;, [events@1.0.2](&quot;https://github.com/Gozala/events&quot;) - &quot;MIT&quot;, [exit@0.1.2](&quot;https://github.com/cowboy/node-exit&quot;) - [&quot;MIT&quot;], [expand-brackets@0.1.4](&quot;git+https://github.com/jonschlinkert/expand-brackets&quot;) - &quot;MIT&quot;, [expand-range@1.8.1](&quot;https://github.com/jonschlinkert/expand-range&quot;) - &quot;MIT&quot;, [express-session@1.11.3](&quot;git+https://github.com/expressjs/session&quot;) - &quot;MIT&quot;, [express@4.13.3](&quot;git+https://github.com/strongloop/express&quot;) - &quot;MIT&quot;, [extend@2.0.1](&quot;git+https://github.com/justmoon/node-extend&quot;) - &quot;MIT&quot;, [extend@3.0.0](&quot;git+https://github.com/justmoon/node-extend&quot;) - &quot;MIT&quot;, [extglob@0.3.1](&quot;https://github.com/jonschlinkert/extglob&quot;) - &quot;MIT&quot;, [extract-text-webpack-plugin@0.8.2](&quot;git+ssh://git@github.com/webpack/extract-text-webpack-plugin&quot;) - &quot;MIT&quot;, [extsprintf@1.2.0](&quot;https://github.com/davepacheco/node-extsprintf&quot;) - &quot;MIT&quot;, [fast-levenshtein@1.0.7](&quot;git+https://github.com/hiddentao/fast-levenshtein&quot;) - &quot;MIT&quot;, [fastparse@1.1.1](&quot;git+https://github.com/webpack/fastparse&quot;) - &quot;MIT&quot;, [figures@1.3.5](&quot;git+https://github.com/sindresorhus/figures&quot;) - &quot;MIT&quot;, [file-loader@0.8.4](&quot;git+https://github.com/webpack/file-loader&quot;) - &quot;MIT&quot;, [file-set@0.2.7](&quot;git+https://github.com/75lb/file-set&quot;) - , [filename-regex@2.0.0](&quot;https://github.com/regexps/filename-regex&quot;) - &quot;MIT&quot;, [fileset@0.2.1](&quot;https://github.com/mklabs/node-fileset&quot;) - [&quot;MIT&quot;], [fill-range@2.2.2](&quot;https://github.com/jonschlinkert/fill-range&quot;) - &quot;MIT&quot;, [finalhandler@0.4.0](&quot;git+https://github.com/pillarjs/finalhandler&quot;) - &quot;MIT&quot;, [find-index@0.1.1](&quot;https://github.com/jsdf/find-index&quot;) - &quot;MIT&quot;, [findup-sync@0.2.1](&quot;https://github.com/cowboy/node-findup-sync&quot;) - [&quot;MIT&quot;], [first-chunk-stream@1.0.0](&quot;git+https://github.com/sindresorhus/first-chunk-stream&quot;) - &quot;MIT&quot;, [flagged-respawn@0.3.1](&quot;https://github.com/tkellen/node-flagged-respawn&quot;) - [&quot;MIT&quot;], [flatten@0.0.1](&quot;https://github.com/jesusabdullah/node-flatten&quot;) - &quot;MIT*&quot;, [for-in@0.1.4](&quot;https://github.com/jonschlinkert/for-in&quot;) - &quot;MIT&quot;, [for-own@0.1.3](&quot;https://github.com/jonschlinkert/for-own&quot;) - &quot;MIT&quot;, [foreach@2.0.5](&quot;https://github.com/manuelstofer/foreach&quot;) - &quot;MIT&quot;, [forever-agent@0.6.1](&quot;git+https://github.com/mikeal/forever-agent&quot;) - &quot;Apache-2.0&quot;, [form-data@1.0.0-rc2](&quot;https://github.com/felixge/node-form-data&quot;) - &quot;MIT&quot;, [form-data@1.0.0-rc3](&quot;https://github.com/form-data/form-data&quot;) - &quot;MIT&quot;, [forwarded@0.1.0](&quot;git+https://github.com/jshttp/forwarded&quot;) - &quot;MIT&quot;, [fresh@0.3.0](&quot;git+https://github.com/jshttp/fresh&quot;) - &quot;MIT&quot;, [fs-extra@0.14.0](&quot;git+https://github.com/jprichardson/node-fs-extra&quot;) - [&quot;MIT&quot;], [fs-readdir-recursive@0.1.2](&quot;git+https://github.com/fs-utils/fs-readdir-recursive&quot;) - &quot;MIT&quot;, [fs-vacuum@1.2.6](&quot;git+https://github.com/npm/fs-vacuum&quot;) - &quot;ISC&quot;, [fs-write-stream-atomic@1.0.3](&quot;https://github.com/npm/fs-write-stream-atomic&quot;) - &quot;ISC&quot;, [fstream-ignore@1.0.2](&quot;https://github.com/isaacs/fstream-ignore&quot;) - &quot;ISC&quot;, [fstream-npm@1.0.4](&quot;https://github.com/isaacs/fstream-npm&quot;) - &quot;ISC&quot;, [fstream@1.0.7](&quot;https://github.com/isaacs/fstream&quot;) - &quot;ISC&quot;, [gather-stream@1.0.0](&quot;https://github.com/natevw/gather-stream&quot;) - &quot;ISC&quot;, [gauge@1.2.0](&quot;git+https://github.com/iarna/gauge&quot;) - &quot;ISC&quot;, [gaze@0.5.1](&quot;https://github.com/shama/gaze&quot;) - [&quot;MIT&quot;], [generate-function@2.0.0](&quot;git+https://github.com/mafintosh/generate-function&quot;) - &quot;MIT&quot;, [generate-object-property@1.2.0](&quot;git+https://github.com/mafintosh/generate-object-property&quot;) - &quot;MIT&quot;, [get-stdin@3.0.2](&quot;git+https://github.com/sindresorhus/get-stdin&quot;) - &quot;MIT&quot;, [get-stdin@4.0.1](&quot;git+https://github.com/sindresorhus/get-stdin&quot;) - &quot;MIT&quot;, [github-url-from-git@1.4.0](&quot;https://github.com/visionmedia/node-github-url-from-git&quot;) - &quot;MIT&quot;, [github-url-from-username-repo@1.0.2](&quot;git@github.com:robertkowalski/github-url-from-username-repo&quot;) - &quot;BSD-2-Clause&quot;, [glob-base@0.2.0](&quot;https://github.com/jonschlinkert/glob-base&quot;) - &quot;MIT&quot;, [glob-parent@1.2.0](&quot;git+https://github.com/es128/glob-parent&quot;) - &quot;ISC&quot;, [glob-stream@3.1.18](&quot;https://github.com/wearefractal/glob-stream&quot;) - [&quot;MIT&quot;], [glob-watcher@0.0.6](&quot;https://github.com/wearefractal/glob-watcher&quot;) - [&quot;MIT&quot;], [glob2base@0.0.12](&quot;https://github.com/wearefractal/glob2base&quot;) - [&quot;MIT&quot;], [glob@3.1.21](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;BSD&quot;, [glob@3.2.11](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;BSD&quot;, [glob@3.2.3](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;BSD&quot;, [glob@4.2.2](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;ISC&quot;, [glob@4.3.5](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;ISC&quot;, [glob@4.5.3](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;ISC&quot;, [glob@5.0.14](&quot;https://github.com/isaacs/node-glob&quot;) - &quot;ISC&quot;, [global@2.0.1](&quot;https://github.com/component/global&quot;) - &quot;MIT&quot;, [globals@6.4.1](&quot;git+https://github.com/sindresorhus/globals&quot;) - &quot;MIT&quot;, [globals@8.6.0](&quot;git+https://github.com/sindresorhus/globals&quot;) - &quot;MIT&quot;, [globule@0.1.0](&quot;https://github.com/cowboy/node-globule&quot;) - [&quot;MIT&quot;], [graceful-fs@1.2.3](&quot;https://github.com/isaacs/node-graceful-fs&quot;) - &quot;BSD&quot;, [graceful-fs@2.0.3](&quot;https://github.com/isaacs/node-graceful-fs&quot;) - &quot;BSD&quot;, [graceful-fs@3.0.8](&quot;https://github.com/isaacs/node-graceful-fs&quot;) - &quot;ISC&quot;, [graceful-fs@4.1.2](&quot;git+https://github.com/isaacs/node-graceful-fs&quot;) - &quot;ISC&quot;, [graceful-readlink@1.0.1](&quot;https://github.com/zhiyelee/graceful-readlink&quot;) - &quot;MIT&quot;, [growl@1.8.1](&quot;https://github.com/visionmedia/node-growl&quot;) - &quot;MIT*&quot;, [gulp-async-func-runner@0.1.9](&quot;https://github.com/Cellarise/gulp-async-func-runner&quot;) - &quot;MIT License (MIT)&quot;, [gulp-concat@2.6.0](&quot;git+https://github.com/wearefractal/gulp-concat&quot;) - &quot;MIT&quot;, [gulp-dust-compile-render@0.2.6](&quot;https://github.com/Cellarise/gulp-dust-compile-render&quot;) - &quot;MIT License (MIT)&quot;, [gulp-eslint@1.0.0](&quot;git+https://github.com/adametry/gulp-eslint&quot;) - &quot;MIT&quot;, [gulp-istanbul@0.10.0](&quot;https://github.com/SBoudrias/gulp-istanbul&quot;) - [&quot;MIT&quot;], [gulp-jsdoc-to-markdown@1.1.1](&quot;git+https://github.com/jsdoc2md/gulp-jsdoc-to-markdown&quot;) - , [gulp-json-editor@2.2.1](&quot;https://github.com/morou/gulp-json-editor&quot;) - &quot;MIT&quot;, [gulp-load-params@0.1.7](&quot;git+https://github.com/Cellarise/gulp-load-params&quot;) - &quot;MIT License (MIT)&quot;, [gulp-mocha@2.1.3](&quot;git+https://github.com/sindresorhus/gulp-mocha&quot;) - &quot;MIT&quot;, [gulp-rename@1.2.2](&quot;https://github.com/hparra/gulp-rename&quot;) - [&quot;MIT&quot;], [gulp-util@3.0.6](&quot;git+https://github.com/wearefractal/gulp-util&quot;) - &quot;MIT&quot;, [gulp-webpack@1.5.0](&quot;git+https://github.com/shama/gulp-webpack&quot;) - &quot;MIT&quot;, [gulp-yadda-steps@0.1.13](&quot;git+https://github.com/Cellarise/gulp-yadda-steps&quot;) - &quot;MIT License (MIT)&quot;, [gulp@3.9.0](&quot;git+https://github.com/gulpjs/gulp&quot;) - &quot;MIT&quot;, [handlebars-ansi@0.2.0](&quot;git+https://github.com/75lb/handlebars-ansi&quot;) - , [handlebars-array@0.2.0](&quot;git+https://github.com/75lb/handlebars-array&quot;) - , [handlebars-comparison@2.0.0](&quot;git+https://github.com/75lb/handlebars-comparison&quot;) - , [handlebars-json@1.0.0](&quot;git+https://github.com/75lb/handlebars-json&quot;) - , [handlebars-regexp@1.0.0](&quot;git+https://github.com/75lb/handlebars-regexp&quot;) - , [handlebars-string@2.0.1](&quot;git+https://github.com/75lb/handlebars-string&quot;) - , [handlebars@3.0.0](&quot;git+https://github.com/wycats/handlebars.js&quot;) - &quot;MIT&quot;, [handlebars@3.0.3](&quot;https://github.com/wycats/handlebars.js&quot;) - &quot;MIT&quot;, [har-validator@1.8.0](&quot;git+https://github.com/ahmadnassri/har-validator&quot;) - &quot;ISC&quot;, [has-ansi@2.0.0](&quot;git+https://github.com/sindresorhus/has-ansi&quot;) - &quot;MIT&quot;, [has-binary-data@0.1.3]([object Object]) - &quot;ISC&quot;, [has-binary@0.1.5]([object Object]) - &quot;MIT&quot;, [has-binary@0.1.6]([object Object]) - &quot;MIT&quot;, [has-cors@1.0.3](&quot;https://github.com/component/has-cors&quot;) - &quot;MIT&quot;, [has-flag@1.0.0](&quot;git+https://github.com/sindresorhus/has-flag&quot;) - &quot;MIT&quot;, [has-unicode@1.0.0](&quot;git+https://github.com/iarna/has-unicode&quot;) - &quot;ISC&quot;, [hawk@3.1.0](&quot;https://github.com/hueniverse/hawk&quot;) - &quot;BSD-3-Clause&quot;, [hoek@2.14.0](&quot;https://github.com/hapijs/hoek&quot;) - &quot;BSD-3-Clause&quot;, [home-or-tmp@1.0.0](&quot;git+https://github.com/sindresorhus/home-or-tmp&quot;) - &quot;MIT&quot;, [hosted-git-info@2.1.4](&quot;git+https://github.com/npm/hosted-git-info&quot;) - &quot;ISC&quot;, [http-browserify@1.7.0](&quot;http://github.com/substack/http-browserify&quot;) - &quot;MIT/X11&quot;, [http-errors@1.3.1](&quot;git+https://github.com/jshttp/http-errors&quot;) - &quot;MIT&quot;, [http-proxy@1.11.1](&quot;git+https://github.com/nodejitsu/node-http-proxy&quot;) - &quot;MIT&quot;, [http-signature@0.11.0](&quot;https://github.com/joyent/node-http-signature&quot;) - &quot;MIT&quot;, [https-browserify@0.0.0](&quot;https://github.com/substack/https-browserify&quot;) - &quot;MIT&quot;, [iconv-lite@0.4.11](&quot;https://github.com/ashtuchkin/iconv-lite&quot;) - &quot;MIT&quot;, [ieee754@1.1.6](&quot;https://github.com/feross/ieee754&quot;) - &quot;MIT&quot;, [image-size@0.3.5](&quot;git@github.com:netroy/image-size&quot;) - &quot;MIT&quot;, [imports-loader@0.6.4](&quot;git+https://github.com/webpack/imports-loader&quot;) - &quot;MIT&quot;, [indent-string@1.2.2](&quot;git+https://github.com/sindresorhus/indent-string&quot;) - &quot;MIT&quot;, [indexes-of@1.0.1](&quot;https://github.com/dominictarr/indexes-of&quot;) - &quot;MIT&quot;, [indexof@0.0.1]([object Object]) - &quot;MIT*&quot;, [inflight@1.0.4](&quot;https://github.com/isaacs/inflight&quot;) - &quot;ISC&quot;, [inherits@1.0.0](&quot;https://github.com/isaacs/inherits&quot;) - , [inherits@2.0.1](&quot;https://github.com/isaacs/inherits&quot;) - &quot;ISC&quot;, [ini@1.3.4](&quot;https://github.com/isaacs/ini&quot;) - &quot;ISC&quot;, [init-package-json@1.8.0](&quot;https://github.com/isaacs/init-package-json&quot;) - &quot;ISC&quot;, [inquirer@0.9.0](&quot;git+https://github.com/sboudrias/Inquirer.js&quot;) - &quot;MIT&quot;, [install@0.1.8](&quot;https://github.com/benjamn/install&quot;) - &quot;MIT*&quot;, [interpret@0.6.5](&quot;https://github.com/tkellen/node-interpret&quot;) - &quot;MIT&quot;, [ipaddr.js@1.0.1](&quot;https://github.com/whitequark/ipaddr.js&quot;) - &quot;MIT&quot;, [is-absolute-url@2.0.0](&quot;git+https://github.com/sindresorhus/is-absolute-url&quot;) - &quot;MIT&quot;, [is-absolute@0.1.7](&quot;https://github.com/jonschlinkert/is-absolute&quot;) - &quot;MIT&quot;, [is-array@1.0.1](&quot;https://github.com/retrofox/is-array&quot;) - &quot;MIT&quot;, [is-binary-path@1.0.1](&quot;git+https://github.com/sindresorhus/is-binary-path&quot;) - &quot;MIT&quot;, [is-dotfile@1.0.1](&quot;https://github.com/jonschlinkert/is-dotfile&quot;) - &quot;MIT&quot;, [is-equal-shallow@0.1.3](&quot;https://github.com/jonschlinkert/is-equal-shallow&quot;) - &quot;MIT&quot;, [is-extglob@1.0.0](&quot;git+https://github.com/jonschlinkert/is-extglob&quot;) - &quot;MIT&quot;, [is-finite@1.0.1](&quot;git+https://github.com/sindresorhus/is-finite&quot;) - &quot;MIT&quot;, [is-glob@1.1.3](&quot;git+https://github.com/jonschlinkert/is-glob&quot;) - &quot;MIT&quot;, [is-integer@1.0.4](&quot;git+ssh://git@github.com/parshap/js-is-integer&quot;) - &quot;MIT&quot;, [is-my-json-valid@2.12.1](&quot;git+https://github.com/mafintosh/is-my-json-valid&quot;) - &quot;MIT&quot;, [is-nan@1.2.1](&quot;https://github.com/ljharb/is-nan&quot;) - &quot;MIT&quot;, [is-number@1.1.2](&quot;https://github.com/jonschlinkert/is-number&quot;) - &quot;MIT&quot;, [is-plain-obj@1.0.0](&quot;git+https://github.com/sindresorhus/is-plain-obj&quot;) - &quot;MIT&quot;, [is-primitive@2.0.0](&quot;https://github.com/jonschlinkert/is-primitive&quot;) - &quot;MIT&quot;, [is-property@1.0.2](&quot;https://github.com/mikolalysenko/is-property&quot;) - &quot;MIT&quot;, [is-relative@0.1.3](&quot;https://github.com/jonschlinkert/is-relative&quot;) - [&quot;MIT&quot;], [is-resolvable@1.0.0](&quot;git+https://github.com/shinnn/is-resolvable&quot;) - &quot;MIT&quot;, [is-utf8@0.2.0](&quot;git+https://github.com/wayfind/is-utf8&quot;) - &quot;BSD&quot;, [isarray@0.0.1](&quot;https://github.com/juliangruber/isarray&quot;) - &quot;MIT&quot;, [isobject@1.0.2](&quot;https://github.com/jonschlinkert/isobject&quot;) - &quot;MIT&quot;, [isstream@0.1.2](&quot;git+https://github.com/rvagg/isstream&quot;) - &quot;MIT&quot;, [istanbul-instrumenter-loader@0.1.3](&quot;git+https://github.com/deepsweet/istanbul-instrumenter-loader&quot;) - &quot;WTFPL&quot;, [istanbul-reporter-clover-limits@0.1.7](&quot;https://github.com/Cellarise/istanbul-reporter-clover-limits&quot;) - &quot;MIT License (MIT)&quot;, [istanbul-threshold-checker@0.1.0](&quot;git+https://github.com/peterjwest/istanbul-threshold-checker&quot;) - &quot;MIT&quot;, [istanbul@0.3.18](&quot;https://github.com/gotwarlost/istanbul&quot;) - &quot;BSD-3-Clause&quot;, [jade@0.26.3](&quot;https://github.com/visionmedia/jade&quot;) - &quot;MIT*&quot;, [javascript-natural-sort@0.7.1](&quot;git+https://github.com/Bill4Time/javascript-natural-sort&quot;) - &quot;MIT&quot;, [jju@1.2.0](&quot;https://github.com/rlidwka/jju&quot;) - &quot;WTFPL&quot;, [js-base64@2.1.9](&quot;https://github.com/dankogai/js-base64&quot;) - &quot;BSD&quot;, [js-beautify@1.5.10](&quot;https://github.com/beautify-web/js-beautify&quot;) - &quot;MIT&quot;, [js-tokens@1.0.1](&quot;git+https://github.com/lydell/js-tokens&quot;) - &quot;MIT&quot;, [js-yaml@3.4.0](&quot;https://github.com/nodeca/js-yaml&quot;) - &quot;MIT&quot;, [js2xmlparser@1.0.0](&quot;https://github.com/michaelkourlas/node-js2xmlparser&quot;) - &quot;Apache-2.0&quot;, [jsdoc-75lb@3.4.0-dev.8](&quot;git+https://github.com/jsdoc2md/jsdoc&quot;) - &quot;Apache-2.0&quot;, [jsdoc-parse@1.0.0](&quot;git+https://github.com/jsdoc2md/jsdoc-parse&quot;) - , [jsdoc-to-markdown@1.1.1](&quot;git+https://github.com/jsdoc2md/jsdoc-to-markdown&quot;) - &quot;MIT*&quot;, [jsesc@0.5.0](&quot;git+https://github.com/mathiasbynens/jsesc&quot;) - [&quot;MIT&quot;], [json-loader@0.5.2](&quot;git+https://github.com/webpack/json-loader&quot;) - &quot;MIT&quot;, [json-parse-helpfulerror@1.0.3](&quot;https://github.com/smikes/json-parse-helpfulerror&quot;) - &quot;MIT&quot;, [json-stringify-safe@5.0.1](&quot;https://github.com/isaacs/json-stringify-safe&quot;) - &quot;ISC&quot;, [json3@3.2.6](&quot;https://github.com/bestiejs/json3&quot;) - [&quot;MIT&quot;], [json5@0.4.0](&quot;git+https://github.com/aseemk/json5&quot;) - &quot;MIT&quot;, [jsonfile@2.2.1](&quot;git+ssh://git@github.com/jprichardson/node-jsonfile&quot;) - &quot;MIT&quot;, [jsonpointer@1.1.0](&quot;git+ssh://git@github.com/janl/node-jsonpointer&quot;) - &quot;MIT*&quot;, [jstransform@10.1.0](&quot;git+ssh://git@github.com/facebook/jstransform&quot;) - [&quot;Apache-2.0&quot;], [kind-of@1.1.0](&quot;https://github.com/jonschlinkert/kind-of&quot;) - &quot;MIT&quot;, [lazy-cache@0.2.3](&quot;git+https://github.com/jonschlinkert/lazy-cache&quot;) - &quot;MIT&quot;, [leadfoot@1.5.2](&quot;git+https://github.com/theintern/leadfoot&quot;) - [&quot;BSD-3-Clause&quot;], [left-pad@0.0.3](&quot;git+ssh://git@github.com/azer/left-pad&quot;) - &quot;BSD&quot;, [less-loader@2.2.0](&quot;https://github.com/webpack/less-loader&quot;) - [&quot;MIT&quot;], [less@2.5.1](&quot;git+https://github.com/less/less.js&quot;) - &quot;Apache-2.0&quot;, [leven@1.0.2](&quot;git+https://github.com/sindresorhus/leven&quot;) - &quot;MIT&quot;, [levn@0.2.5](&quot;https://github.com/gkz/levn&quot;) - [&quot;MIT&quot;], [liftoff@2.1.0](&quot;https://github.com/tkellen/node-liftoff&quot;) - [&quot;MIT&quot;], [line-numbers@0.2.0](&quot;git+https://github.com/lydell/line-numbers&quot;) - &quot;MIT&quot;, [loader-utils@0.2.11](&quot;git+https://github.com/webpack/loader-utils&quot;) - &quot;MIT&quot;, [lockfile@1.0.1](&quot;https://github.com/isaacs/lockfile&quot;) - &quot;ISC&quot;, [lodash._arraycopy@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._arrayeach@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._arraymap@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._baseassign@3.2.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._baseclone@3.3.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basecopy@3.0.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basedifference@3.0.3](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._baseflatten@3.1.4](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basefor@3.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._baseindexof@3.1.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basetostring@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basetostring@3.0.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._basevalues@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._bindcallback@3.0.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._cacheindexof@3.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._createassigner@3.1.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._createcache@3.1.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._createpadding@3.6.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._getnative@3.9.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._isiterateecall@3.0.9](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._pickbyarray@3.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._pickbycallback@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._reescape@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._reevaluate@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash._reinterpolate@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.clonedeep@3.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.escape@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.isarguments@3.0.4](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.isarray@3.0.4](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.isplainobject@3.2.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.istypedarray@3.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.keys@3.1.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.keysin@3.0.8](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.merge@3.3.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.omit@3.1.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.pad@3.1.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.padleft@3.1.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.padright@3.1.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.repeat@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.restparam@3.6.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.template@3.6.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.templatesettings@3.1.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash.toplainobject@3.0.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash@1.0.2](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash@3.10.1](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lodash@3.6.0](&quot;git+https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [lru-cache@2.6.5](&quot;https://github.com/isaacs/node-lru-cache&quot;) - &quot;ISC&quot;, [lru-queue@0.1.0](&quot;https://github.com/medikoo/lru-queue&quot;) - &quot;MIT&quot;, [map-obj@1.0.1](&quot;git+https://github.com/sindresorhus/map-obj&quot;) - &quot;MIT&quot;, [marked@0.3.5](&quot;https://github.com/chjj/marked&quot;) - &quot;MIT&quot;, [media-typer@0.3.0](&quot;git+https://github.com/jshttp/media-typer&quot;) - &quot;MIT&quot;, [memoizee@0.3.9](&quot;https://github.com/medikoo/memoize&quot;) - &quot;MIT&quot;, [memory-fs@0.2.0](&quot;git+https://github.com/webpack/memory-fs&quot;) - &quot;MIT&quot;, [meow@3.3.0](&quot;git+https://github.com/sindresorhus/meow&quot;) - &quot;MIT&quot;, [merge-descriptors@1.0.0](&quot;git+https://github.com/component/merge-descriptors&quot;) - &quot;MIT&quot;, [methods@1.1.1](&quot;git+https://github.com/jshttp/methods&quot;) - &quot;MIT&quot;, [micromatch@2.2.0](&quot;git+https://github.com/jonschlinkert/micromatch&quot;) - &quot;MIT&quot;, [mime-db@1.15.0](&quot;git+https://github.com/jshttp/mime-db&quot;) - &quot;MIT&quot;, [mime-db@1.17.0](&quot;git+https://github.com/jshttp/mime-db&quot;) - &quot;MIT&quot;, [mime-types@2.1.3](&quot;git+https://github.com/jshttp/mime-types&quot;) - &quot;MIT&quot;, [mime-types@2.1.5](&quot;git+https://github.com/jshttp/mime-types&quot;) - &quot;MIT&quot;, [mime@1.2.11](&quot;git+https://github.com/broofa/node-mime&quot;) - &quot;MIT*&quot;, [mime@1.3.4](&quot;git+https://github.com/broofa/node-mime&quot;) - [&quot;MIT&quot;], [minimatch@0.2.14](&quot;https://github.com/isaacs/minimatch&quot;) - &quot;MIT&quot;, [minimatch@0.3.0](&quot;https://github.com/isaacs/minimatch&quot;) - &quot;MIT&quot;, [minimatch@1.0.0](&quot;https://github.com/isaacs/minimatch&quot;) - &quot;MIT&quot;, [minimatch@2.0.10](&quot;https://github.com/isaacs/minimatch&quot;) - &quot;ISC&quot;, [minimatch@2.0.8](&quot;https://github.com/isaacs/minimatch&quot;) - &quot;ISC&quot;, [minimist@0.0.10](&quot;https://github.com/substack/minimist&quot;) - &quot;MIT&quot;, [minimist@0.0.8](&quot;https://github.com/substack/minimist&quot;) - &quot;MIT&quot;, [minimist@1.1.3](&quot;https://github.com/substack/minimist&quot;) - &quot;MIT&quot;, [mkdirp@0.3.0](&quot;https://github.com/substack/node-mkdirp&quot;) - &quot;MIT/X11&quot;, [mkdirp@0.5.0](&quot;git+https://github.com/substack/node-mkdirp&quot;) - &quot;MIT&quot;, [mkdirp@0.5.1](&quot;git+https://github.com/substack/node-mkdirp&quot;) - &quot;MIT&quot;, [mocha-bamboo-reporter-bgo@1.0.7](&quot;https://github.com/issacg/mocha-bamboo-reporter&quot;) - [&quot;Apache 2.0&quot;], [mocha@2.2.5](&quot;https://github.com/mochajs/mocha&quot;) - &quot;MIT&quot;, [more-fs@0.5.0](&quot;git+https://github.com/75lb/more-fs&quot;) - , [morgan@1.6.1](&quot;git+https://github.com/expressjs/morgan&quot;) - &quot;MIT&quot;, [ms@0.6.2](&quot;https://github.com/guille/ms.js&quot;) - &quot;MIT*&quot;, [ms@0.7.1](&quot;https://github.com/guille/ms.js&quot;) - &quot;MIT*&quot;, [multipipe@0.1.2](&quot;git+https://github.com/juliangruber/multipipe&quot;) - &quot;MIT&quot;, [mute-stream@0.0.4](&quot;https://github.com/isaacs/mute-stream&quot;) - &quot;BSD&quot;, [mute-stream@0.0.5](&quot;https://github.com/isaacs/mute-stream&quot;) - &quot;ISC&quot;, [mv@2.1.1](&quot;https://github.com/andrewrk/node-mv&quot;) - &quot;MIT&quot;, [nan@1.8.4](&quot;https://github.com/iojs/nan&quot;) - &quot;MIT&quot;, [nature@0.5.7](&quot;git+https://github.com/75lb/nature&quot;) - &quot;MIT&quot;, [ncp@1.0.1](&quot;git+https://github.com/AvianFlu/ncp&quot;) - &quot;MIT&quot;, [ncp@2.0.0](&quot;git+https://github.com/AvianFlu/ncp&quot;) - &quot;MIT&quot;, [negotiator@0.5.3](&quot;git+https://github.com/jshttp/negotiator&quot;) - &quot;MIT&quot;, [next-tick@0.2.2](&quot;https://github.com/medikoo/next-tick&quot;) - &quot;MIT&quot;, [node-balanced@0.0.14](&quot;https://github.com/icodeforlove/node-balanced&quot;) - &quot;MIT&quot;, [node-gyp@2.0.2](&quot;https://github.com/TooTallNate/node-gyp&quot;) - &quot;MIT&quot;, [node-libs-browser@0.5.2](&quot;git+https://github.com/webpack/node-libs-browser&quot;) - &quot;MIT&quot;, [node-uuid@1.4.3](&quot;git+https://github.com/broofa/node-uuid&quot;) - [&quot;MIT&quot;], [nopt-usage@0.1.0]([object Object]) - &quot;MIT&quot;, [nopt@3.0.3](&quot;git+ssh://git@github.com/isaacs/nopt&quot;) - &quot;ISC&quot;, [normalize-git-url@3.0.1](&quot;git+https://github.com/npm/normalize-git-url&quot;) - &quot;ISC&quot;, [normalize-package-data@1.0.3](&quot;https://github.com/meryn/normalize-package-data&quot;) - &quot;MIT*&quot;, [normalize-package-data@2.3.1](&quot;https://github.com/npm/normalize-package-data&quot;) - &quot;BSD-2-Clause&quot;, [normalize-selector@0.2.0](&quot;https://github.com/getify/normalize-selector&quot;) - &quot;MIT&quot;, [normalize-url@1.3.0](&quot;git+https://github.com/sindresorhus/normalize-url&quot;) - &quot;MIT&quot;, [npm-cache-filename@1.0.2](&quot;https://github.com/npm/npm-cache-filename&quot;) - &quot;ISC&quot;, [npm-install-checks@1.0.6](&quot;https://github.com/npm/npm-install-checks&quot;) - &quot;BSD-2-Clause&quot;, [npm-license@0.3.1](&quot;http://github.com/AceMetrix/npm-license&quot;) - [&quot;BSD&quot;], [npm-package-arg@4.0.2](&quot;git+https://github.com/npm/npm-package-arg&quot;) - &quot;ISC&quot;, [npm-registry-client@7.0.1](&quot;https://github.com/isaacs/npm-registry-client&quot;) - &quot;ISC&quot;, [npm-user-validate@0.1.2](&quot;https://github.com/npm/npm-user-validate&quot;) - &quot;BSD-2-Clause&quot;, [npm@2.14.0](&quot;git+https://github.com/npm/npm&quot;) - &quot;Artistic-2.0&quot;, [npmlog@1.2.1](&quot;https://github.com/isaacs/npmlog&quot;) - &quot;ISC&quot;, [num2fraction@1.1.0](&quot;git+ssh://git@github.com/yisibl/num2fraction&quot;) - &quot;MIT&quot;, [number-is-nan@1.0.0](&quot;git+https://github.com/sindresorhus/number-is-nan&quot;) - &quot;MIT&quot;, [oauth-rest-atlassian@0.4.19](&quot;git+https://github.com/Cellarise/OAuth-REST-Atlassian&quot;) - &quot;MIT License (MIT)&quot;, [oauth-sign@0.8.0](&quot;git+https://github.com/mikeal/oauth-sign&quot;) - &quot;Apache-2.0&quot;, [object-assign@2.1.1](&quot;git+https://github.com/sindresorhus/object-assign&quot;) - &quot;MIT&quot;, [object-assign@3.0.0](&quot;git+https://github.com/sindresorhus/object-assign&quot;) - &quot;MIT&quot;, [object-component@0.0.3]([object Object]) - &quot;MIT*&quot;, [object-keys@0.4.0](&quot;https://github.com/ljharb/object-keys&quot;) - &quot;MIT&quot;, [object-keys@1.0.1](&quot;https://github.com/ljharb/object-keys&quot;) - &quot;MIT&quot;, [object-keys@1.0.7](&quot;https://github.com/ljharb/object-keys&quot;) - &quot;MIT&quot;, [object-tools@1.6.7](&quot;git+https://github.com/75lb/object-tools&quot;) - &quot;MIT&quot;, [object.omit@1.1.0](&quot;https://github.com/jonschlinkert/object.omit&quot;) - &quot;MIT&quot;, [on-finished@2.3.0](&quot;git+https://github.com/jshttp/on-finished&quot;) - &quot;MIT&quot;, [on-headers@1.0.0](&quot;git+https://github.com/jshttp/on-headers&quot;) - &quot;MIT&quot;, [once@1.3.2](&quot;https://github.com/isaacs/once&quot;) - &quot;ISC&quot;, [opener@1.4.1](&quot;https://github.com/domenic/opener&quot;) - &quot;WTFPL&quot;, [optimist@0.3.7](&quot;git+ssh://git@github.com/substack/node-optimist&quot;) - &quot;MIT/X11&quot;, [optimist@0.6.1](&quot;git+ssh://git@github.com/substack/node-optimist&quot;) - &quot;MIT/X11&quot;, [optionator@0.5.0](&quot;https://github.com/gkz/optionator&quot;) - [&quot;MIT&quot;], [options@0.0.6](&quot;https://github.com/einaros/options.js&quot;) - &quot;MIT*&quot;, [orchestrator@0.3.7](&quot;https://github.com/robrich/orchestrator&quot;) - [&quot;MIT&quot;], [ordered-read-streams@0.1.0](&quot;https://github.com/armed/ordered-read-streams&quot;) - &quot;MIT&quot;, [os-browserify@0.1.2](&quot;http://github.com/drewyoung1/os-browserify&quot;) - &quot;MIT&quot;, [os-homedir@1.0.0](&quot;git+https://github.com/sindresorhus/os-homedir&quot;) - &quot;MIT&quot;, [os-homedir@1.0.1](&quot;git+https://github.com/sindresorhus/os-homedir&quot;) - &quot;MIT&quot;, [os-tmpdir@1.0.1](&quot;git+https://github.com/sindresorhus/os-tmpdir&quot;) - &quot;MIT&quot;, [osenv@0.1.3](&quot;git+https://github.com/npm/osenv&quot;) - &quot;ISC&quot;, [output-file-sync@1.1.1](&quot;git+https://github.com/shinnn/output-file-sync&quot;) - &quot;MIT&quot;, [package-license@0.1.2](&quot;https://github.com/AceMetrix/package-license&quot;) - &quot;Apache2&quot;, [pako@0.2.7](&quot;https://github.com/nodeca/pako&quot;) - &quot;MIT&quot;, [parse-glob@3.0.2](&quot;https://github.com/jonschlinkert/parse-glob&quot;) - &quot;MIT&quot;, [parsejson@0.0.1]([object Object]) - &quot;MIT&quot;, [parseqs@0.0.2]([object Object]) - &quot;MIT&quot;, [parseuri@0.0.2]([object Object]) - &quot;MIT&quot;, [parseuri@0.0.4](&quot;git+https://github.com/get/parseuri&quot;) - &quot;MIT&quot;, [parseurl@1.3.0](&quot;git+https://github.com/expressjs/parseurl&quot;) - &quot;MIT&quot;, [path-array@1.0.0](&quot;https://github.com/TooTallNate/node-path-array&quot;) - &quot;MIT&quot;, [path-browserify@0.0.0](&quot;https://github.com/substack/path-browserify&quot;) - &quot;MIT&quot;, [path-exists@1.0.0](&quot;git+https://github.com/sindresorhus/path-exists&quot;) - &quot;MIT&quot;, [path-is-absolute@1.0.0](&quot;git+https://github.com/sindresorhus/path-is-absolute&quot;) - &quot;MIT&quot;, [path-is-inside@1.0.1](&quot;https://github.com/domenic/path-is-inside&quot;) - &quot;WTFPL&quot;, [path-to-regexp@0.1.7](&quot;git+https://github.com/component/path-to-regexp&quot;) - &quot;MIT&quot;, [pbkdf2-compat@2.0.1](&quot;https://github.com/dcousens/pbkdf2-compat&quot;) - &quot;MIT&quot;, [pkginfo@0.3.0](&quot;http://github.com/indexzero/node-pkginfo&quot;) - &quot;MIT*&quot;, [plur@1.0.0](&quot;git+https://github.com/sindresorhus/plur&quot;) - &quot;MIT&quot;, [postcss-calc@4.1.0](&quot;git+https://github.com/postcss/postcss-calc&quot;) - &quot;MIT&quot;, [postcss-colormin@1.2.5](&quot;git+https://github.com/ben-eb/postcss-colormin&quot;) - &quot;MIT&quot;, [postcss-convert-values@1.2.5](&quot;git+https://github.com/ben-eb/postcss-convert-values&quot;) - &quot;MIT&quot;, [postcss-discard-comments@1.2.1](&quot;https://github.com/ben-eb/postcss-discard-comments&quot;) - &quot;MIT&quot;, [postcss-discard-duplicates@1.1.5](&quot;git+https://github.com/ben-eb/postcss-discard-duplicates&quot;) - &quot;MIT&quot;, [postcss-discard-empty@1.1.2](&quot;git+https://github.com/ben-eb/postcss-discard-empty&quot;) - &quot;MIT&quot;, [postcss-discard-unused@1.0.3](&quot;git+https://github.com/ben-eb/postcss-discard-unused&quot;) - &quot;MIT&quot;, [postcss-filter-plugins@1.0.0](&quot;git+https://github.com/postcss/postcss-filter-plugins&quot;) - &quot;MIT&quot;, [postcss-font-family@1.2.1](&quot;https://github.com/ben-eb/postcss-font-family&quot;) - &quot;MIT&quot;, [postcss-merge-idents@1.0.1](&quot;git+https://github.com/ben-eb/postcss-merge-idents&quot;) - &quot;MIT&quot;, [postcss-merge-longhand@1.0.2](&quot;git+https://github.com/ben-eb/postcss-merge-longhand&quot;) - &quot;MIT&quot;, [postcss-merge-rules@1.3.5](&quot;git+https://github.com/ben-eb/postcss-merge-rules&quot;) - &quot;MIT&quot;, [postcss-message-helpers@2.0.0](&quot;git+https://github.com/MoOx/postcss-message-helpers&quot;) - &quot;MIT&quot;, [postcss-minify-font-weight@1.0.1](&quot;git+https://github.com/ben-eb/postcss-minify-font-weight&quot;) - &quot;MIT&quot;, [postcss-minify-selectors@1.5.0](&quot;https://github.com/ben-eb/postcss-minify-selectors&quot;) - &quot;MIT&quot;, [postcss-modules-extract-imports@0.0.5](&quot;git+https://github.com/css-modules/postcss-modules-extract-imports&quot;) - &quot;ISC&quot;, [postcss-modules-local-by-default@0.0.11](&quot;git+https://github.com/css-modules/postcss-modules-local-by-default&quot;) - &quot;MIT&quot;, [postcss-modules-scope@0.0.8](&quot;git+https://github.com/geelen/postcss-modules-scope&quot;) - &quot;ISC&quot;, [postcss-normalize-url@2.1.1](&quot;git+https://github.com/ben-eb/postcss-normalize-url&quot;) - &quot;MIT&quot;, [postcss-ordered-values@1.1.0](&quot;git+https://github.com/ben-eb/postcss-ordered-values&quot;) - &quot;MIT&quot;, [postcss-reduce-idents@1.0.2](&quot;git+https://github.com/ben-eb/postcss-reduce-idents&quot;) - &quot;MIT&quot;, [postcss-selector-parser@1.1.4](&quot;git+https://github.com/postcss/postcss-selector-parser&quot;) - &quot;MIT&quot;, [postcss-single-charset@0.3.0](&quot;git+https://github.com/hail2u/postcss-single-charset&quot;) - &quot;MIT&quot;, [postcss-unique-selectors@1.0.0](&quot;git+https://github.com/ben-eb/postcss-unique-selectors&quot;) - &quot;MIT&quot;, [postcss-value-parser@1.1.0](&quot;git+https://github.com/TrySound/postcss-value-parser&quot;) - &quot;MIT&quot;, [postcss-zindex@1.1.3](&quot;git+https://github.com/ben-eb/postcss-zindex&quot;) - &quot;MIT&quot;, [postcss@4.1.16](&quot;git+https://github.com/postcss/postcss&quot;) - &quot;MIT&quot;, [prelude-ls@1.1.2](&quot;https://github.com/gkz/prelude-ls&quot;) - [&quot;MIT&quot;], [prepend-http@1.0.2](&quot;git+https://github.com/sindresorhus/prepend-http&quot;) - &quot;MIT&quot;, [preserve@0.2.0](&quot;https://github.com/jonschlinkert/preserve&quot;) - &quot;MIT&quot;, [pretty-hrtime@1.0.0](&quot;https://github.com/robrich/pretty-hrtime&quot;) - [&quot;MIT&quot;], [private@0.1.6](&quot;https://github.com/benjamn/private&quot;) - &quot;MIT&quot;, [process-nextick-args@1.0.2](&quot;git+https://github.com/calvinmetcalf/process-nextick-args&quot;) - &quot;MIT&quot;, [process@0.11.1](&quot;https://github.com/shtylman/node-process&quot;) - &quot;MIT*&quot;, [promise@6.1.0](&quot;git+https://github.com/then/promise&quot;) - &quot;MIT&quot;, [promzard@0.3.0](&quot;https://github.com/isaacs/promzard&quot;) - &quot;ISC&quot;, [proto-list@1.2.4](&quot;git+https://github.com/isaacs/proto-list&quot;) - &quot;ISC&quot;, [proxy-addr@1.0.8](&quot;git+https://github.com/jshttp/proxy-addr&quot;) - &quot;MIT&quot;, [prr@0.0.0](&quot;https://github.com/rvagg/prr&quot;) - &quot;MIT&quot;, [punycode@1.3.2](&quot;git+https://github.com/bestiejs/punycode.js&quot;) - &quot;MIT&quot;, [q@1.1.2](&quot;https://github.com/kriskowal/q&quot;) - &quot;MIT&quot;, [qs@4.0.0](&quot;git+https://github.com/hapijs/qs&quot;) - &quot;BSD-3-Clause&quot;, [query-string@2.4.0](&quot;git+https://github.com/sindresorhus/query-string&quot;) - &quot;MIT&quot;, [querystring-es3@0.2.1](&quot;https://github.com/mike-spainhower/querystring&quot;) - [&quot;MIT&quot;], [querystring@0.2.0](&quot;https://github.com/Gozala/querystring&quot;) - [&quot;MIT&quot;], [ramda@0.14.0](&quot;https://github.com/ramda/ramda&quot;) - &quot;MIT&quot;, [ramda@0.17.1](&quot;https://github.com/ramda/ramda&quot;) - &quot;MIT&quot;, [randomatic@1.1.0](&quot;git+https://github.com/jonschlinkert/randomatic&quot;) - &quot;MIT&quot;, [range-parser@1.0.2](&quot;git+https://github.com/jshttp/range-parser&quot;) - &quot;MIT&quot;, [raw-loader@0.5.1](&quot;git@github.com:webpack/raw-loader&quot;) - [&quot;MIT&quot;], [react-hot-api@0.4.6](&quot;git+https://github.com/gaearon/react-hot-api&quot;) - &quot;MIT&quot;, [react-hot-loader@1.2.9](&quot;git+https://github.com/gaearon/react-hot-loader&quot;) - &quot;MIT&quot;, [react@0.13.3](&quot;git+https://github.com/facebook/react&quot;) - &quot;BSD-3-Clause&quot;, [read-file-stdin@0.2.0](&quot;https://github.com/ianstormtaylor/read-file-stdin&quot;) - &quot;MIT&quot;, [read-installed@3.1.5](&quot;https://github.com/isaacs/read-installed&quot;) - &quot;ISC&quot;, [read-installed@4.0.2](&quot;https://github.com/isaacs/read-installed&quot;) - &quot;ISC&quot;, [read-package-json@1.3.3](&quot;https://github.com/isaacs/read-package-json&quot;) - &quot;ISC&quot;, [read-package-json@2.0.0](&quot;https://github.com/isaacs/read-package-json&quot;) - &quot;ISC&quot;, [read@1.0.6](&quot;https://github.com/isaacs/read&quot;) - &quot;ISC&quot;, [readable-stream@1.0.33](&quot;https://github.com/isaacs/readable-stream&quot;) - &quot;MIT&quot;, [readable-stream@1.1.13](&quot;https://github.com/isaacs/readable-stream&quot;) - &quot;MIT&quot;, [readable-stream@2.0.2](&quot;https://github.com/nodejs/readable-stream&quot;) - &quot;MIT&quot;, [readdir-scoped-modules@1.0.1](&quot;git+https://github.com/npm/readdir-scoped-modules&quot;) - &quot;ISC&quot;, [readdirp@1.4.0](&quot;https://github.com/thlorenz/readdirp&quot;) - &quot;MIT&quot;, [readline2@0.1.1](&quot;git+https://github.com/SBoudrias/readline2&quot;) - &quot;MIT&quot;, [realize-package-specifier@3.0.1](&quot;git+https://github.com/npm/realize-package-specifier&quot;) - &quot;ISC&quot;, [recast@0.10.24](&quot;https://github.com/benjamn/recast&quot;) - &quot;MIT&quot;, [recast@0.10.32](&quot;https://github.com/benjamn/recast&quot;) - &quot;MIT&quot;, [rechoir@0.6.2](&quot;https://github.com/tkellen/node-rechoir&quot;) - [&quot;MIT&quot;], [reduce-css-calc@1.2.0](&quot;git+https://github.com/MoOx/reduce-css-calc&quot;) - &quot;MIT&quot;, [reduce-function-call@1.0.1](&quot;git+https://github.com/MoOx/reduce-function-call&quot;) - &quot;MIT&quot;, [regenerate@1.2.1](&quot;git+https://github.com/mathiasbynens/regenerate&quot;) - &quot;MIT&quot;, [regenerator@0.8.35](&quot;https://github.com/facebook/regenerator&quot;) - &quot;BSD&quot;, [regex-cache@0.4.2](&quot;https://github.com/jonschlinkert/regex-cache&quot;) - &quot;MIT&quot;, [regexpu@1.2.0](&quot;git+https://github.com/mathiasbynens/regexpu&quot;) - &quot;MIT&quot;, [regjsgen@0.2.0](&quot;https://github.com/d10/regjsgen&quot;) - &quot;MIT&quot;, [regjsparser@0.1.5](&quot;git+ssh://git@github.com/jviereck/regjsparser&quot;) - &quot;BSD&quot;, [repeat-element@1.1.2](&quot;https://github.com/jonschlinkert/repeat-element&quot;) - &quot;MIT&quot;, [repeat-string@1.5.2](&quot;https://github.com/jonschlinkert/repeat-string&quot;) - &quot;MIT&quot;, [repeating@1.1.3](&quot;git+https://github.com/sindresorhus/repeating&quot;) - &quot;MIT&quot;, [replace-ext@0.0.1](&quot;https://github.com/wearefractal/replace-ext&quot;) - [&quot;MIT&quot;], [request@2.60.0](&quot;git+https://github.com/request/request&quot;) - &quot;Apache-2.0&quot;, [request@2.61.0](&quot;git+https://github.com/request/request&quot;) - &quot;Apache-2.0&quot;, [requires-port@0.0.1](&quot;git+https://github.com/unshiftio/requires-port&quot;) - &quot;MIT&quot;, [requizzle@0.2.1](&quot;https://github.com/hegemonic/requizzle&quot;) - &quot;MIT&quot;, [resolve-from@1.0.0](&quot;https://github.com/sindresorhus/resolve-from&quot;) - &quot;MIT&quot;, [resolve@1.1.6](&quot;https://github.com/substack/node-resolve&quot;) - &quot;MIT&quot;, [retry@0.6.1](&quot;https://github.com/tim-kos/node-retry&quot;) - &quot;MIT*&quot;, [rimraf@2.2.8](&quot;https://github.com/isaacs/rimraf&quot;) - &quot;MIT&quot;, [rimraf@2.4.2](&quot;https://github.com/isaacs/rimraf&quot;) - &quot;ISC&quot;, [ripemd160@0.2.0](&quot;https://github.com/cryptocoinjs/ripemd160&quot;) - , [run-async@0.1.0](&quot;https://github.com/SBoudrias/run-async&quot;) - &quot;MIT&quot;, [run-sequence@1.1.2](&quot;https://github.com/OverZealous/run-sequence&quot;) - &quot;MIT&quot;, [rx-lite@2.5.2](&quot;git+https://github.com/Reactive-Extensions/RxJS&quot;) - [&quot;Apache License, Version 2.0&quot;], [safe-json-stringify@1.0.3](&quot;git+ssh://git@github.com/e-conomic/safe-json-stringify&quot;) - &quot;MIT&quot;, [script-loader@0.6.1](&quot;git+ssh://git@github.com/webpack/script-loader&quot;) - [&quot;MIT&quot;], [semver@4.3.6](&quot;https://github.com/npm/node-semver&quot;) - &quot;ISC&quot;, [semver@5.0.1](&quot;git+https://github.com/npm/node-semver&quot;) - &quot;ISC&quot;, [send@0.13.0](&quot;git+https://github.com/pillarjs/send&quot;) - &quot;MIT&quot;, [sequencify@0.0.7](&quot;https://github.com/robrich/sequencify&quot;) - [&quot;MIT&quot;], [serve-index@1.7.2](&quot;git+https://github.com/expressjs/serve-index&quot;) - &quot;MIT&quot;, [serve-static@1.10.0](&quot;git+https://github.com/expressjs/serve-static&quot;) - &quot;MIT&quot;, [sha.js@2.2.6](&quot;https://github.com/dominictarr/sha.js&quot;) - &quot;MIT&quot;, [sha@1.3.0](&quot;https://github.com/ForbesLindesay/sha&quot;) - &quot;BSD&quot;, [shebang-regex@1.0.0](&quot;git+https://github.com/sindresorhus/shebang-regex&quot;) - &quot;MIT&quot;, [should-equal@0.5.0](&quot;git+https://github.com/shouldjs/equal&quot;) - &quot;MIT&quot;, [should-format@0.3.0](&quot;https://github.com/shouldjs/format&quot;) - &quot;MIT&quot;, [should-type@0.2.0](&quot;git+https://github.com/shouldjs/type&quot;) - &quot;MIT&quot;, [should@7.0.4](&quot;git+https://github.com/shouldjs/should.js&quot;) - &quot;MIT&quot;, [sigmund@1.0.1](&quot;https://github.com/isaacs/sigmund&quot;) - &quot;ISC&quot;, [simple-fmt@0.1.0](&quot;git+https://github.com/olov/simple-fmt&quot;) - &quot;MIT&quot;, [simple-is@0.2.0](&quot;git+https://github.com/olov/simple-is&quot;) - &quot;MIT&quot;, [slash@1.0.0](&quot;https://github.com/sindresorhus/slash&quot;) - &quot;MIT&quot;, [slide@1.1.6](&quot;https://github.com/isaacs/slide-flow-control&quot;) - &quot;ISC&quot;, [sloc@0.1.9](&quot;https://github.com/flosse/sloc&quot;) - &quot;MIT&quot;, [sntp@1.0.9](&quot;https://github.com/hueniverse/sntp&quot;) - [&quot;BSD&quot;], [socket.io-adapter@0.3.1](&quot;https://github.com/Automattic/socket.io-adapter&quot;) - &quot;MIT*&quot;, [socket.io-client@1.3.6](&quot;git+https://github.com/Automattic/socket.io-client&quot;) - &quot;MIT&quot;, [socket.io-parser@2.2.2](&quot;git+https://github.com/Automattic/socket.io-parser&quot;) - &quot;MIT&quot;, [socket.io-parser@2.2.4](&quot;git+https://github.com/Automattic/socket.io-parser&quot;) - &quot;MIT&quot;, [socket.io@1.3.6](&quot;https://github.com/Automattic/socket.io&quot;) - &quot;MIT*&quot;, [sort-keys@1.1.1](&quot;git+https://github.com/sindresorhus/sort-keys&quot;) - &quot;MIT&quot;, [sorted-object@1.0.0](&quot;https://github.com/domenic/sorted-object&quot;) - &quot;WTFPL&quot;, [source-list-map@0.1.5](&quot;https://github.com/webpack/source-list-map&quot;) - &quot;MIT&quot;, [source-map-support@0.2.10](&quot;git+https://github.com/evanw/node-source-map-support&quot;) - [&quot;MIT&quot;], [source-map@0.1.31](&quot;git+ssh://git@github.com/mozilla/source-map&quot;) - [&quot;BSD&quot;], [source-map@0.1.32](&quot;git+ssh://git@github.com/mozilla/source-map&quot;) - [&quot;BSD&quot;], [source-map@0.1.34](&quot;git+ssh://git@github.com/mozilla/source-map&quot;) - [&quot;BSD&quot;], [source-map@0.1.43](&quot;git+ssh://git@github.com/mozilla/source-map&quot;) - [&quot;BSD&quot;], [source-map@0.4.4](&quot;git+ssh://git@github.com/mozilla/source-map&quot;) - &quot;BSD-3-Clause&quot;, [spdx-correct@1.0.0](&quot;git+https://github.com/kemitchell/spdx-correct.js&quot;) - &quot;Apache-2.0&quot;, [spdx-license-ids@1.0.1](&quot;git+https://github.com/shinnn/spdx-license-ids&quot;) - &quot;Unlicense&quot;, [spdx@0.4.1](&quot;git+https://github.com/kemitchell/spdx.js&quot;) - &quot;Apache-2.0&quot;, [sprintf-js@1.0.3](&quot;git+https://github.com/alexei/sprintf.js&quot;) - &quot;BSD-3-Clause&quot;, [stable@0.1.5](&quot;git+https://github.com/Two-Screen/stable&quot;) - &quot;MIT&quot;, [statuses@1.2.1](&quot;git+https://github.com/jshttp/statuses&quot;) - &quot;MIT&quot;, [stream-browserify@1.0.0](&quot;https://github.com/substack/stream-browserify&quot;) - &quot;MIT&quot;, [stream-cache@0.0.2](&quot;https://github.com/felixge/node-stream-cache&quot;) - &quot;MIT*&quot;, [stream-connect@0.1.0](&quot;git+https://github.com/75lb/stream-connect&quot;) - &quot;MIT&quot;, [stream-consume@0.1.0](&quot;git+https://github.com/aroneous/stream-consume&quot;) - &quot;MIT&quot;, [stream-handlebars@0.1.6](&quot;git+https://github.com/75lb/stream-handlebars&quot;) - &quot;MIT&quot;, [stream-via@0.1.0](&quot;git+https://github.com/75lb/stream-via&quot;) - &quot;MIT&quot;, [streamifier@0.1.1](&quot;https://github.com/gagle/node-streamifier&quot;) - &quot;MIT&quot;, [strict-uri-encode@1.0.2](&quot;git+https://github.com/kevva/strict-uri-encode&quot;) - &quot;MIT&quot;, [string-tools@0.1.8](&quot;git+https://github.com/75lb/string-tools&quot;) - &quot;MIT&quot;, [string_decoder@0.10.31](&quot;https://github.com/rvagg/string_decoder&quot;) - &quot;MIT&quot;, [stringmap@0.2.2](&quot;git+https://github.com/olov/stringmap&quot;) - &quot;MIT&quot;, [stringset@0.2.1](&quot;git+https://github.com/olov/stringset&quot;) - &quot;MIT&quot;, [stringstream@0.0.4](&quot;git+https://github.com/mhart/StringStream&quot;) - &quot;MIT&quot;, [strip-ansi@2.0.1](&quot;git+https://github.com/sindresorhus/strip-ansi&quot;) - &quot;MIT&quot;, [strip-ansi@3.0.0](&quot;git+https://github.com/sindresorhus/strip-ansi&quot;) - &quot;MIT&quot;, [strip-bom@1.0.0](&quot;https://github.com/sindresorhus/strip-bom&quot;) - &quot;MIT&quot;, [strip-json-comments@1.0.4](&quot;git+https://github.com/sindresorhus/strip-json-comments&quot;) - &quot;MIT&quot;, [style-loader@0.12.3](&quot;git+ssh://git@github.com/webpack/style-loader&quot;) - &quot;MIT&quot;, [success-symbol@0.1.0](&quot;git+https://github.com/jonschlinkert/success-symbol&quot;) - &quot;MIT&quot;, [supports-color@1.2.1](&quot;git+https://github.com/sindresorhus/supports-color&quot;) - &quot;MIT&quot;, [supports-color@1.3.1](&quot;git+https://github.com/sindresorhus/supports-color&quot;) - &quot;MIT&quot;, [supports-color@2.0.0](&quot;git+https://github.com/chalk/supports-color&quot;) - &quot;MIT&quot;, [supports-color@3.1.0](&quot;git+https://github.com/chalk/supports-color&quot;) - &quot;MIT&quot;, [taffydb@2.6.2](&quot;https://github.com/hegemonic/taffydb&quot;) - &quot;BSD&quot;, [tapable@0.1.9]([object Object]) - [&quot;MIT&quot;], [tar@1.0.3](&quot;https://github.com/isaacs/node-tar&quot;) - &quot;BSD&quot;, [tar@2.1.1](&quot;https://github.com/isaacs/node-tar&quot;) - &quot;ISC&quot;, [temp@0.8.3](&quot;https://github.com/bruce/node-temp&quot;) - &quot;MIT&quot;, [text-table@0.2.0](&quot;https://github.com/substack/text-table&quot;) - &quot;MIT&quot;, [through2@0.5.1](&quot;git+https://github.com/rvagg/through2&quot;) - &quot;MIT&quot;, [through2@0.6.5](&quot;git+https://github.com/rvagg/through2&quot;) - &quot;MIT&quot;, [through2@2.0.0](&quot;git+https://github.com/rvagg/through2&quot;) - &quot;MIT&quot;, [through@2.3.8](&quot;git+https://github.com/dominictarr/through&quot;) - &quot;MIT&quot;, [tildify@1.1.0](&quot;git+https://github.com/sindresorhus/tildify&quot;) - &quot;MIT&quot;, [timers-browserify@1.4.1](&quot;https://github.com/jryans/timers-browserify&quot;) - [&quot;MIT&quot;], [timers-ext@0.1.0](&quot;https://github.com/medikoo/timers-ext&quot;) - &quot;MIT&quot;, [to-array@0.1.3](&quot;https://github.com/Raynos/to-array&quot;) - [&quot;MIT&quot;], [to-fast-properties@1.0.1](&quot;git+https://github.com/sindresorhus/to-fast-properties&quot;) - &quot;MIT&quot;, [tough-cookie@2.0.0](&quot;https://github.com/SalesforceEng/tough-cookie&quot;) - &quot;BSD-3-Clause&quot;, [treeify@1.0.1](&quot;https://github.com/notatestuser/treeify&quot;) - [&quot;MIT&quot;], [trim-right@1.0.1](&quot;git+https://github.com/sindresorhus/trim-right&quot;) - &quot;MIT&quot;, [try-resolve@1.0.1](&quot;git+https://github.com/sebmck/try-resolve&quot;) - &quot;MIT&quot;, [tryit@1.0.1](&quot;git+ssh://git@github.com/HenrikJoreteg/tryit&quot;) - &quot;MIT&quot;, [tryor@0.1.2](&quot;git+https://github.com/olov/tryor&quot;) - &quot;MIT&quot;, [tty-browserify@0.0.0](&quot;https://github.com/substack/tty-browserify&quot;) - &quot;MIT&quot;, [tunnel-agent@0.4.1](&quot;git+https://github.com/mikeal/tunnel-agent&quot;) - &quot;Apache*&quot;, [type-check@0.3.1](&quot;https://github.com/gkz/type-check&quot;) - [&quot;MIT&quot;], [type-detect@0.1.1](&quot;git@github.com:chaijs/type-detect&quot;) - &quot;MIT&quot;, [type-detect@1.0.0](&quot;git@github.com:chaijs/type-detect&quot;) - &quot;MIT&quot;, [type-is@1.6.7](&quot;git+https://github.com/jshttp/type-is&quot;) - &quot;MIT&quot;, [typedarray@0.0.6](&quot;https://github.com/substack/typedarray&quot;) - &quot;MIT&quot;, [typical@1.0.0](&quot;git+https://github.com/75lb/typical&quot;) - , [typical@2.3.1](&quot;git+https://github.com/75lb/typical&quot;) - , [uglify-js@2.3.6](&quot;git+https://github.com/mishoo/UglifyJS2&quot;) - &quot;MIT*&quot;, [uglify-js@2.4.24](&quot;git+https://github.com/mishoo/UglifyJS2&quot;) - &quot;BSD&quot;, [uglify-to-browserify@1.0.2](&quot;git+https://github.com/ForbesLindesay/uglify-to-browserify&quot;) - &quot;MIT&quot;, [uid-number@0.0.6](&quot;https://github.com/isaacs/uid-number&quot;) - &quot;ISC&quot;, [uid-safe@2.0.0](&quot;git+https://github.com/crypto-utils/uid-safe&quot;) - &quot;MIT&quot;, [ultron@1.0.2](&quot;git+https://github.com/unshiftio/ultron&quot;) - &quot;MIT&quot;, [umask@1.1.0](&quot;https://github.com/smikes/umask&quot;) - &quot;MIT&quot;, [underscore-contrib@0.3.0](&quot;https://github.com/documentcloud/underscore-contrib&quot;) - &quot;MIT&quot;, [underscore@1.4.4](&quot;https://github.com/documentcloud/underscore&quot;) - &quot;MIT*&quot;, [underscore@1.6.0](&quot;https://github.com/jashkenas/underscore&quot;) - [&quot;MIT&quot;], [underscore@1.8.3](&quot;https://github.com/jashkenas/underscore&quot;) - &quot;MIT&quot;, [uniq@1.0.1](&quot;https://github.com/mikolalysenko/uniq&quot;) - &quot;MIT&quot;, [uniqid@1.0.0](&quot;git+https://github.com/adamhalasz/uniqid&quot;) - &quot;MIT&quot;, [uniqs@2.0.0](&quot;https://github.com/fgnass/uniqs&quot;) - &quot;MIT&quot;, [unique-stream@1.0.0](&quot;git+https://github.com/eugeneware/unique-stream&quot;) - &quot;BSD&quot;, [unpipe@1.0.0](&quot;git+https://github.com/stream-utils/unpipe&quot;) - &quot;MIT&quot;, [url-loader@0.5.6](&quot;git+ssh://git@github.com/webpack/url-loader&quot;) - &quot;MIT&quot;, [url@0.10.3](&quot;https://github.com/defunctzombie/node-url&quot;) - &quot;MIT&quot;, [user-home@1.1.1](&quot;git+https://github.com/sindresorhus/user-home&quot;) - &quot;MIT&quot;, [utf8@2.0.0](&quot;git+https://github.com/mathiasbynens/utf8.js&quot;) - [&quot;MIT&quot;,&quot;GPL&quot;], [util-deprecate@1.0.1](&quot;https://github.com/TooTallNate/util-deprecate&quot;) - &quot;MIT&quot;, [util-extend@1.0.1](&quot;https://github.com/isaacs/util-extend&quot;) - &quot;MIT&quot;, [util@0.10.3](&quot;https://github.com/defunctzombie/node-util&quot;) - &quot;MIT&quot;, [utils-merge@1.0.0](&quot;https://github.com/jaredhanson/utils-merge&quot;) - [&quot;MIT&quot;], [v8flags@2.0.10](&quot;https://github.com/tkellen/node-v8flags&quot;) - [&quot;MIT&quot;], [validate-npm-package-license@2.0.0](&quot;git+https://github.com/kemitchell/validate-npm-package-license.js&quot;) - &quot;Apache-2.0&quot;, [validate-npm-package-name@2.2.2](&quot;git+https://github.com/npm/validate-npm-package-name&quot;) - &quot;ISC&quot;, [vary@1.0.1](&quot;git+https://github.com/jshttp/vary&quot;) - &quot;MIT&quot;, [vasync@1.6.3](&quot;https://github.com/davepacheco/node-vasync&quot;) - &quot;MIT&quot;, [verror@1.6.0](&quot;https://github.com/davepacheco/node-verror&quot;) - &quot;MIT&quot;, [vinyl-fs@0.3.13](&quot;https://github.com/wearefractal/vinyl-fs&quot;) - [&quot;MIT&quot;], [vinyl@0.4.6](&quot;https://github.com/wearefractal/vinyl&quot;) - [&quot;MIT&quot;], [vinyl@0.5.1](&quot;https://github.com/wearefractal/vinyl&quot;) - &quot;MIT&quot;, [vm-browserify@0.0.4](&quot;http://github.com/substack/vm-browserify&quot;) - &quot;MIT&quot;, [watchpack@0.2.8](&quot;git+https://github.com/webpack/watchpack&quot;) - &quot;MIT&quot;, [wcwidth@1.0.0]([object Object]) - &quot;MIT&quot;, [webpack-core@0.6.6](&quot;git+https://github.com/webpack/core&quot;) - [&quot;MIT&quot;], [webpack-dev-middleware@1.2.0](&quot;git+https://github.com/webpack/webpack-dev-middleware&quot;) - [&quot;MIT&quot;], [webpack-dev-server@1.10.1](&quot;https://github.com/webpack/webpack-dev-server&quot;) - &quot;MIT&quot;, [webpack@1.11.0](&quot;git+ssh://git@github.com/webpack/webpack&quot;) - &quot;MIT&quot;, [which@1.0.9](&quot;https://github.com/isaacs/node-which&quot;) - &quot;ISC&quot;, [which@1.1.1](&quot;https://github.com/isaacs/node-which&quot;) - &quot;ISC&quot;, [window-size@0.1.0](&quot;git+https://github.com/jonschlinkert/window-size&quot;) - [&quot;MIT&quot;], [word-wrap@1.1.0](&quot;git+https://github.com/jonschlinkert/word-wrap&quot;) - &quot;MIT&quot;, [wordwrap@0.0.2](&quot;https://github.com/substack/node-wordwrap&quot;) - &quot;MIT/X11&quot;, [wordwrap@0.0.3](&quot;https://github.com/substack/node-wordwrap&quot;) - &quot;MIT&quot;, [wrappy@1.0.1](&quot;git+https://github.com/npm/wrappy&quot;) - &quot;ISC&quot;, [wrench@1.5.8](&quot;https://ryanmcgrath@github.com/ryanmcgrath/wrench-js&quot;) - [&quot;MIT&quot;], [write-file-atomic@1.1.2](&quot;git@github.com:iarna/write-file-atomic&quot;) - &quot;ISC&quot;, [write-file-stdout@0.0.2](&quot;https://github.com/ianstormtaylor/write-file-stdout&quot;) - &quot;MIT&quot;, [ws@0.7.2](&quot;https://github.com/websockets/ws&quot;) - &quot;MIT&quot;, [xml-escape@1.0.0](&quot;https://github.com/miketheprogrammer/xml-escape&quot;) - &quot;MIT License&quot;, [xmlhttprequest@1.5.0](&quot;https://github.com/driverdan/node-XMLHttpRequest&quot;) - [&quot;MIT&quot;], [xtend@2.1.2](&quot;https://github.com/Raynos/xtend&quot;) - [&quot;MIT&quot;], [xtend@3.0.0](&quot;https://github.com/Raynos/xtend&quot;) - [&quot;MIT&quot;], [xtend@4.0.0](&quot;https://github.com/Raynos/xtend&quot;) - [&quot;MIT&quot;], [yadda@0.11.7](&quot;https://github.com/acuminous/yadda&quot;) - &quot;Apache2&quot;, [yadda@0.15.2](&quot;https://github.com/acuminous/yadda&quot;) - &quot;Apache2&quot;, [yargs@1.3.3](&quot;git+ssh://git@github.com/chevex/yargs&quot;) - &quot;MIT/X11&quot;, [yargs@3.5.4](&quot;git+ssh://git@github.com/bcoe/yargs&quot;) - &quot;MIT/X11&quot;, 
*documented by [npm-licenses](http://github.com/AceMetrix/npm-license.git)*.
