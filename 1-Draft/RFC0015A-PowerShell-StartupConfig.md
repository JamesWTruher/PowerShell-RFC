---
RFC: RFC0015-A
Author: James Truher
Status: Draft
Version: 0.1
Area: Startup, Initial Configuration
Comments Due: 7/15/2017
---

# Startup Configuration Settings for PowerShell Core

Full PowerShell uses the registry to store configuration data which controls many different behaviors when PowerShell starts.
A number of different locations in the registry are used to store config which makes it difficult to manage and retrieve configuration.
This is obviously a problem for non-Windows systems which require system wide configuration, but the registry is not present.

## Motivation

* As a user, I would like to be sure that I do not have to set additional configuration (such as execution policy) when my shell starts.
* As an admin, I would like to be able to control the execution policy and other settings of the shell as it starts.
* As a tester, I often need to have configuration set _before_ the PowerShell engine is running to enable certain test behaviors.

The registry exists only on Windows, non-Windows platforms still need a mechanism for applying system configuration.
Additionally, since PowerShell Core may have multiple installations on a single system, configuration must be specific to the version installed.

## Specification

The file $PSHOME/PowerShell.Config shall contain the configuration to be used at startup, the format of which is a simple text file with key=value pairs
If the file $PSHOME/PowerShell.Config is found, the configuration therein will be read and a data structure shall be created which may be used by features needing configuration data.
This file shall be read-only, e.g., current tools which currently write to these locations in the registry will return an error if used.
Also, changes shall be read only at startup time, changes made to the file will not affect any running sessions.



* InternalTestHooks
    * This allows test configuration to be set before the PowerShell engine has started
* PowerShell
    * This allows for configuration of ExecutionPolicy, ConsolePrompting, DisablePromptForUpdateHelp, PSModulePath

Additional keys and subkeys may be added as needed.
Keys which are not supported shall generate a _warning_ and be ignored.

The configuration file in $PSHOME shall be applicable only to the PowerShell executable found therein.
The configuration file in $HOME (PowerShell._version_.Config.psd) shall be applicable to only to the matching version of PowerShell Core.
Additionally, the parameter `-ConfigurationFile` shall be present in the PowerShell executable, which takes an array of strings which represent paths to config files.
When the `-ConfigurationFile` parameter is present, those files only will be read for configuration data.

The config file should have a published schema to aid in file creation.

### Additional Possibilities
* A global PowerShell.Config.psd1 file could be placed in a common location (such as /etc or %ALLUSERSPROFILE%) which could be applicable to all PowerShell sessions, but that is not in scope for this proposal.
* Tools to ease the creation/alteration/removal of settings should be created at a future date.

### Additional Optional Work
We should have a mechanism for creating default config files, we could easily modify existing code for New-PSSessionConfiguraitonFile to do this, however this is not a requirement.

### Sample configuration file
```

# Test settings
LogPipelineCommandToFile=1
PipelineLogFile=c:/tmp/command.log
StopwatchIsNotHighResolution=0
ForceScriptBlockLogging=0
GciEnumerationActionDelete=0
GciEnumerationActionRename=0
IgnoreScriptBlockCache=0
UseDebugAmsiImplementation=0

# PowerShell behavior settings
ConsolePrompting=1
DisablePromptToUpdateHelp=0
ExecutionPolicy=Unrestricted
PipelineMaxStackSizeMB=10
UpdatableHelp.DefaultSourcePath=/tmp
Transcription=0
ModuleLogging=0
ScriptBlockLogging=0
# do not send telemetry set 0
Telemetry=1


```

## Out of Scope
This specification is applicable only to startup preferences and does not apply to setting Policy.
Setting Policy behavior, such as found via GroupPolicy, may be addressed in a later specification, if desired.

## Alternate Proposals and Considerations

### EnvironmentalVariables used to set configuration
This has a number of problems, the foremost is that environment variables may be changed or even deleted by the user.
The current proposal provides for those settings to be unalterable by a user which does not have permission to alter the file contents.

### Datafile Formats
The format for the datafile could take many forms; XML, json, Linux Config, .yml, or .ini.
I chose PowerShell Data format assuming that anyone that is configuring PowerShell is familiar with PowerShell constructs, and code is available to convert a PSD1 file to an object before the engine is started.
Also, PSD1 files may conin comments, which may be very helpful in describing the settings.
The config file location is somewhat forced due to PowerShell's side-by-side requirements, so a global location (.e.g., `/etc/`) would greatly complicate the configuration file to cover multiple versions.

