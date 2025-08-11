# Topic: IniRead Function - Configuration File Management and Settings Retrieval

## Category

Function

## Overview

The IniRead function retrieves values from INI configuration files, providing essential configuration management capabilities for applications requiring persistent settings, user preferences, and multi-environment configuration support. It's crucial for application configuration, user customization, deployment settings, enterprise configuration management, and any system requiring structured data persistence.

## Key Points

- Reads configuration values from INI files with comprehensive error handling and default value support
- Supports hierarchical configuration with sections, keys, and nested configuration management
- Provides foundation for application settings, user preferences, and environment-specific configuration
- Enables sophisticated configuration frameworks including inheritance, validation, and dynamic reloading
- Essential for enterprise applications, deployment automation, user customization, and multi-environment systems

## Syntax and Parameters

```cpp
Value := IniRead(Filename, Section, Key [, Default])

; Parameters:
; Filename - Path to INI file (relative or absolute)
; Section  - Section name within the INI file
; Key      - Key name within the specified section
; Default  - Default value if key/section not found (optional, default: empty string)

; INI file format:
; [SectionName]
; KeyName=Value
; AnotherKey=AnotherValue
;
; [AnotherSection]
; Setting1=Value1
; Setting2=Value2

; Return values:
; String value from INI file
; Default value if key/section doesn't exist
; Empty string if no default specified and key not found

; File path handling:
; Relative paths resolved relative to working directory
; Use A_ScriptDir for paths relative to script location
; Supports UNC paths and network locations
; Creates file automatically on first write (via IniWrite)

; Error handling:
; Returns default value for missing keys/sections
; File access errors return default value
; Invalid file format returns default value
; No exceptions thrown - always returns a string value
```

## Code Examples

```cpp
; Basic INI reading
value := IniRead("config.ini", "Settings", "Username", "DefaultUser")

; Read with specific default
timeout := IniRead("app.ini", "Network", "Timeout", "30")

; Read from script directory
configPath := A_ScriptDir . "\config.ini"
theme := IniRead(configPath, "UI", "Theme", "Light")

; Advanced configuration management system
class ConfigManager {
    static configs := Map()
    static watchers := Map()
    static defaultConfig := Map()
    static environments := ["development", "staging", "production"]
    static currentEnvironment := "development"
    static configPath := A_ScriptDir . "\config"
    static autoReload := true
    static reloadInterval := 5000
    static validationRules := Map()
    
    static Initialize() {
        this.configs.Clear()
        this.watchers.Clear()
        this.LoadDefaultConfiguration()
        this.StartFileWatching()
    }
    
    static LoadConfiguration(configName, environment := "") {
        ; Load configuration with environment support
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        ; Build file paths
        baseConfigFile := this.configPath . "\" . configName . ".ini"
        envConfigFile := this.configPath . "\" . configName . "." . environment . ".ini"
        
        ; Create configuration object
        config := {
            name: configName,
            environment: environment,
            baseFile: baseConfigFile,
            envFile: envConfigFile,
            values: Map(),
            lastModified: 0,
            loaded: false,
            errors: []
        }
        
        try {
            ; Load base configuration
            if (FileExist(baseConfigFile)) {
                this.LoadIniFile(config, baseConfigFile, false)
            }
            
            ; Load environment-specific overrides
            if (FileExist(envConfigFile)) {
                this.LoadIniFile(config, envConfigFile, true)
            }
            
            ; Apply default values for missing keys
            this.ApplyDefaults(config)
            
            ; Validate configuration
            this.ValidateConfiguration(config)
            
            config.loaded := true
            config.lastModified := A_TickCount
            
        } catch Error as err {
            config.errors.Push({
                type: "load_error",
                message: err.Message,
                timestamp: A_TickCount
            })
        }
        
        ; Store configuration
        configKey := configName . ":" . environment
        this.configs[configKey] := config
        
        return config
    }
    
    static LoadIniFile(config, filePath, isOverride := false) {
        ; Read entire INI file and parse into configuration
        
        if (!FileExist(filePath)) {
            return false
        }
        
        try {
            content := FileRead(filePath)
            lines := StrSplit(content, "`n", "`r")
            
            currentSection := ""
            
            for line in lines {
                line := Trim(line)
                
                ; Skip empty lines and comments
                if (!line || SubStr(line, 1, 1) = ";" || SubStr(line, 1, 1) = "#") {
                    continue
                }
                
                ; Parse section headers
                if (RegExMatch(line, "^\[(.+)\]$", &match)) {
                    currentSection := match[1]
                    continue
                }
                
                ; Parse key-value pairs
                if (InStr(line, "=") && currentSection) {
                    parts := StrSplit(line, "=", " `t", 2)
                    if (parts.Length = 2) {
                        key := Trim(parts[1])
                        value := Trim(parts[2])
                        
                        ; Remove quotes if present
                        if ((SubStr(value, 1, 1) = '"' && SubStr(value, -1) = '"') ||
                            (SubStr(value, 1, 1) = "'" && SubStr(value, -1) = "'")) {
                            value := SubStr(value, 2, -1)
                        }
                        
                        ; Store in nested structure
                        sectionKey := currentSection . "." . key
                        
                        ; Track overrides
                        if (isOverride && config.values.Has(sectionKey)) {
                            if (!config.HasProp("overrides")) {
                                config.overrides := Map()
                            }
                            config.overrides[sectionKey] := {
                                original: config.values[sectionKey],
                                override: value,
                                source: filePath
                            }
                        }
                        
                        config.values[sectionKey] := value
                    }
                }
            }
            
            return true
            
        } catch Error as err {
            config.errors.Push({
                type: "parse_error",
                file: filePath,
                message: err.Message,
                timestamp: A_TickCount
            })
            return false
        }
    }
    
    static GetValue(configName, section, key, defaultValue := "", environment := "") {
        ; Get configuration value with fallback logic
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        configKey := configName . ":" . environment
        
        ; Load configuration if not already loaded
        if (!this.configs.Has(configKey)) {
            this.LoadConfiguration(configName, environment)
        }
        
        config := this.configs[configKey]
        valueKey := section . "." . key
        
        ; Return value if found
        if (config.values.Has(valueKey)) {
            return this.ConvertValue(config.values[valueKey])
        }
        
        ; Check default configuration
        if (this.defaultConfig.Has(valueKey)) {
            return this.ConvertValue(this.defaultConfig[valueKey])
        }
        
        ; Return provided default
        return defaultValue
    }
    
    static ConvertValue(value) {
        ; Convert string values to appropriate types
        if (value = "true" || value = "True" || value = "TRUE") {
            return true
        } else if (value = "false" || value = "False" || value = "FALSE") {
            return false
        } else if (IsInteger(value)) {
            return Integer(value)
        } else if (IsFloat(value)) {
            return Float(value)
        } else {
            return value
        }
    }
    
    static GetSection(configName, section, environment := "") {
        ; Get all values from a configuration section
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        configKey := configName . ":" . environment
        
        if (!this.configs.Has(configKey)) {
            this.LoadConfiguration(configName, environment)
        }
        
        config := this.configs[configKey]
        sectionValues := Map()
        sectionPrefix := section . "."
        
        ; Find all keys in the section
        for key, value in config.values {
            if (SubStr(key, 1, StrLen(sectionPrefix)) = sectionPrefix) {
                keyName := SubStr(key, StrLen(sectionPrefix) + 1)
                sectionValues[keyName] := this.ConvertValue(value)
            }
        }
        
        return sectionValues
    }
    
    static GetAllSections(configName, environment := "") {
        ; Get all sections from configuration
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        configKey := configName . ":" . environment
        
        if (!this.configs.Has(configKey)) {
            this.LoadConfiguration(configName, environment)
        }
        
        config := this.configs[configKey]
        sections := Map()
        
        ; Group values by section
        for key, value in config.values {
            if (InStr(key, ".")) {
                parts := StrSplit(key, ".", , 2)
                sectionName := parts[1]
                keyName := parts[2]
                
                if (!sections.Has(sectionName)) {
                    sections[sectionName] := Map()
                }
                
                sections[sectionName][keyName] := this.ConvertValue(value)
            }
        }
        
        return sections
    }
    
    static SetDefault(section, key, value) {
        ; Set default value for configuration key
        defaultKey := section . "." . key
        this.defaultConfig[defaultKey] := value
    }
    
    static LoadDefaultConfiguration() {
        ; Load application default configuration
        this.SetDefault("Application", "Name", "AutoHotkey Application")
        this.SetDefault("Application", "Version", "1.0.0")
        this.SetDefault("Application", "Debug", false)
        
        this.SetDefault("UI", "Theme", "Light")
        this.SetDefault("UI", "Language", "English")
        this.SetDefault("UI", "WindowWidth", 800)
        this.SetDefault("UI", "WindowHeight", 600)
        
        this.SetDefault("Network", "Timeout", 30)
        this.SetDefault("Network", "Retries", 3)
        this.SetDefault("Network", "UseProxy", false)
        
        this.SetDefault("Logging", "Level", "INFO")
        this.SetDefault("Logging", "MaxFileSize", 10485760)  ; 10MB
        this.SetDefault("Logging", "MaxFiles", 5)
    }
    
    static ReloadConfiguration(configName, environment := "") {
        ; Reload configuration from disk
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        configKey := configName . ":" . environment
        
        ; Remove existing configuration
        if (this.configs.Has(configKey)) {
            this.configs.Delete(configKey)
        }
        
        ; Reload from files
        return this.LoadConfiguration(configName, environment)
    }
    
    static SetEnvironment(environment) {
        ; Change current environment
        if (!this.environments.Find(environment)) {
            throw ValueError("Invalid environment: " . environment)
        }
        
        this.currentEnvironment := environment
        
        ; Reload all configurations for new environment
        loadedConfigs := []
        for configKey in this.configs {
            parts := StrSplit(configKey, ":")
            if (parts.Length = 2) {
                loadedConfigs.Push(parts[1])
            }
        }
        
        ; Clear and reload
        this.configs.Clear()
        for configName in loadedConfigs {
            this.LoadConfiguration(configName, environment)
        }
    }
    
    static StartFileWatching() {
        ; Monitor configuration files for changes
        if (!this.autoReload) return
        
        SetTimer(() => this.CheckFileChanges(), this.reloadInterval)
    }
    
    static CheckFileChanges() {
        ; Check if configuration files have changed
        if (!this.autoReload) return
        
        for configKey, config in this.configs {
            changed := false
            
            ; Check base file
            if (FileExist(config.baseFile)) {
                fileTime := FileGetTime(config.baseFile, "M")
                if (fileTime > config.lastModified) {
                    changed := true
                }
            }
            
            ; Check environment file
            if (FileExist(config.envFile)) {
                fileTime := FileGetTime(config.envFile, "M")
                if (fileTime > config.lastModified) {
                    changed := true
                }
            }
            
            ; Reload if changed
            if (changed) {
                this.ReloadConfiguration(config.name, config.environment)
                this.NotifyConfigurationChange(config.name, config.environment)
            }
        }
    }
    
    static NotifyConfigurationChange(configName, environment) {
        ; Override this method to implement change notifications
        if (this.HasProp("onConfigurationChange") && IsFunc(this.onConfigurationChange)) {
            this.onConfigurationChange.Call(configName, environment)
        }
    }
    
    static AddValidationRule(section, key, rule) {
        ; Add validation rule for configuration value
        ruleKey := section . "." . key
        this.validationRules[ruleKey] := rule
    }
    
    static ValidateConfiguration(config) {
        ; Validate configuration against defined rules
        for ruleKey, rule in this.validationRules {
            if (config.values.Has(ruleKey)) {
                value := config.values[ruleKey]
                
                try {
                    if (!this.ValidateValue(value, rule)) {
                        config.errors.Push({
                            type: "validation_error",
                            key: ruleKey,
                            value: value,
                            rule: rule,
                            timestamp: A_TickCount
                        })
                    }
                } catch Error as err {
                    config.errors.Push({
                        type: "validation_exception",
                        key: ruleKey,
                        message: err.Message,
                        timestamp: A_TickCount
                    })
                }
            }
        }
    }
    
    static ValidateValue(value, rule) {
        ; Validate individual value against rule
        switch rule.type {
            case "integer":
                if (!IsInteger(value)) return false
                if (rule.HasProp("min") && Integer(value) < rule.min) return false
                if (rule.HasProp("max") && Integer(value) > rule.max) return false
                
            case "float":
                if (!IsFloat(value) && !IsInteger(value)) return false
                if (rule.HasProp("min") && Float(value) < rule.min) return false
                if (rule.HasProp("max") && Float(value) > rule.max) return false
                
            case "string":
                if (rule.HasProp("minLength") && StrLen(value) < rule.minLength) return false
                if (rule.HasProp("maxLength") && StrLen(value) > rule.maxLength) return false
                if (rule.HasProp("pattern") && !RegExMatch(value, rule.pattern)) return false
                
            case "enum":
                if (rule.HasProp("values") && !rule.values.Find(value)) return false
                
            case "boolean":
                if (value != "true" && value != "false" && value != true && value != false) return false
                
            case "path":
                if (rule.HasProp("mustExist") && rule.mustExist && !FileExist(value)) return false
                
            default:
                return true
        }
        
        return true
    }
    
    static GetConfigurationInfo(configName, environment := "") {
        ; Get detailed information about configuration
        if (!environment) {
            environment := this.currentEnvironment
        }
        
        configKey := configName . ":" . environment
        
        if (!this.configs.Has(configKey)) {
            return {}
        }
        
        config := this.configs[configKey]
        
        return {
            name: config.name,
            environment: config.environment,
            loaded: config.loaded,
            lastModified: config.lastModified,
            valueCount: config.values.Count,
            errorCount: config.errors.Length,
            baseFileExists: FileExist(config.baseFile),
            envFileExists: FileExist(config.envFile),
            hasOverrides: config.HasProp("overrides") && config.overrides.Count > 0
        }
    }
    
    static GenerateConfigurationReport() {
        ; Generate comprehensive configuration report
        report := "Configuration Management Report`n"
        report .= "================================`n`n"
        
        report .= "Environment: " . this.currentEnvironment . "`n"
        report .= "Auto Reload: " . (this.autoReload ? "Enabled" : "Disabled") . "`n"
        report .= "Total Configurations: " . this.configs.Count . "`n`n"
        
        ; Default configuration
        report .= "Default Configuration:`n"
        for key, value in this.defaultConfig {
            report .= "  " . key . " = " . value . "`n"
        }
        report .= "`n"
        
        ; Loaded configurations
        report .= "Loaded Configurations:`n"
        for configKey, config in this.configs {
            report .= "  " . configKey . ":`n"
            report .= "    Values: " . config.values.Count . "`n"
            report .= "    Errors: " . config.errors.Length . "`n"
            report .= "    Base File: " . (FileExist(config.baseFile) ? "Found" : "Missing") . "`n"
            report .= "    Env File: " . (FileExist(config.envFile) ? "Found" : "Missing") . "`n"
            
            if (config.errors.Length > 0) {
                report .= "    Recent Errors:`n"
                for error in config.errors {
                    report .= "      " . error.type . ": " . error.message . "`n"
                }
            }
            report .= "`n"
        }
        
        return report
    }
    
    static EnableAutoReload() {
        this.autoReload := true
        this.StartFileWatching()
    }
    
    static DisableAutoReload() {
        this.autoReload := false
    }
}

; Example usage and demonstrations
; Initialize configuration system
ConfigManager.Initialize()

; Basic INI reading examples
username := IniRead("config.ini", "User", "Name", "Guest")
timeout := IniRead("network.ini", "Connection", "Timeout", "30")
debugMode := IniRead("app.ini", "Debug", "Enabled", "false")

; Advanced configuration management
; Add validation rules
ConfigManager.AddValidationRule("Network", "Timeout", {type: "integer", min: 5, max: 300})
ConfigManager.AddValidationRule("UI", "Theme", {type: "enum", values: ["Light", "Dark", "Auto"]})
ConfigManager.AddValidationRule("Logging", "Level", {type: "enum", values: ["DEBUG", "INFO", "WARN", "ERROR"]})

; Load application configuration
appConfig := ConfigManager.LoadConfiguration("application", "production")

; Get configuration values
dbHost := ConfigManager.GetValue("application", "Database", "Host", "localhost")
dbPort := ConfigManager.GetValue("application", "Database", "Port", 5432)
uiTheme := ConfigManager.GetValue("application", "UI", "Theme", "Light")

; Get entire sections
databaseConfig := ConfigManager.GetSection("application", "Database")
for key, value in databaseConfig {
    OutputDebug("DB " . key . ": " . value)
}

; Environment switching
ConfigManager.SetEnvironment("staging")
stagingDbHost := ConfigManager.GetValue("application", "Database", "Host", "localhost")

; Configuration monitoring
ConfigManager.onConfigurationChange := (configName, environment) => {
    OutputDebug("Configuration changed: " . configName . " (" . environment . ")")
    ; Reload application components that depend on configuration
}

; Enable automatic configuration reloading
ConfigManager.EnableAutoReload()

; Generate configuration reports
configReport := ConfigManager.GenerateConfigurationReport()
; MsgBox(configReport, "Configuration Report")

; Multi-environment configuration example
for env in ["development", "staging", "production"] {
    ConfigManager.SetEnvironment(env)
    
    ; Get environment-specific values
    dbHost := ConfigManager.GetValue("application", "Database", "Host")
    logLevel := ConfigManager.GetValue("application", "Logging", "Level")
    
    OutputDebug(env . " - DB Host: " . dbHost . ", Log Level: " . logLevel)
}

; Configuration validation example
ConfigManager.SetEnvironment("development")
devConfig := ConfigManager.LoadConfiguration("application", "development")

if (devConfig.errors.Length > 0) {
    MsgBox("Configuration errors found:")
    for error in devConfig.errors {
        MsgBox("  " . error.type . ": " . error.message)
    }
}

; Complex configuration structure example
; Reading nested configuration values
serverConfig := Map(
    "web_port", ConfigManager.GetValue("application", "Server", "WebPort", 8080),
    "api_port", ConfigManager.GetValue("application", "Server", "ApiPort", 8081),
    "ssl_enabled", ConfigManager.GetValue("application", "Server", "SSL", false),
    "max_connections", ConfigManager.GetValue("application", "Server", "MaxConnections", 100)
)

; Database connection configuration
dbConfig := Map(
    "host", ConfigManager.GetValue("application", "Database", "Host", "localhost"),
    "port", ConfigManager.GetValue("application", "Database", "Port", 5432),
    "database", ConfigManager.GetValue("application", "Database", "Name", "app_db"),
    "username", ConfigManager.GetValue("application", "Database", "Username", "user"),
    "password", ConfigManager.GetValue("application", "Database", "Password", ""),
    "pool_size", ConfigManager.GetValue("application", "Database", "PoolSize", 10)
)

; Feature flags configuration
features := Map(
    "new_ui", ConfigManager.GetValue("application", "Features", "NewUI", false),
    "beta_features", ConfigManager.GetValue("application", "Features", "BetaFeatures", false),
    "analytics", ConfigManager.GetValue("application", "Features", "Analytics", true),
    "debug_mode", ConfigManager.GetValue("application", "Features", "DebugMode", false)
)

; Dynamic configuration reloading example
ReloadConfiguration() {
    ; Reload all configurations
    ConfigManager.ReloadConfiguration("application")
    
    ; Notify user
    ToolTip("Configuration reloaded", 10, 10)
    SetTimer(() => ToolTip(), -2000)
}

; Set up hotkey for manual configuration reload
Hotkey("Ctrl+Alt+R", ReloadConfiguration)

; Configuration file watching with notifications
if (ConfigManager.autoReload) {
    ToolTip("Configuration auto-reload enabled", 10, 30)
    SetTimer(() => ToolTip(), -3000)
}
```

## Implementation Notes

**File Handling and Performance:**
- INI file reading is synchronous and can block for large files or slow storage
- Cache frequently accessed configuration values to reduce file I/O operations
- Consider file locking mechanisms for configuration files shared between multiple processes
- Monitor file system performance when implementing automatic reload functionality

**Configuration Structure and Organization:**
- Use consistent section and key naming conventions across all configuration files
- Implement hierarchical configuration with base files and environment-specific overrides
- Consider configuration inheritance and cascading for complex multi-environment setups
- Document configuration schema and provide validation for critical settings

**Data Type Handling:**
- INI files store all values as strings, requiring conversion for non-string data types
- Implement robust type conversion with error handling for invalid values
- Consider JSON or XML formats for complex data structures requiring native type support
- Validate configuration values against expected types and ranges to prevent runtime errors

**Security and Sensitive Data:**
- Avoid storing sensitive information like passwords directly in plain text INI files
- Implement encryption or use secure configuration stores for sensitive settings
- Consider file permissions and access control for configuration files containing sensitive data
- Use environment variables or secure vaults for production secret management

**Error Handling and Reliability:**
- Always provide meaningful default values for configuration settings
- Implement graceful degradation when configuration files are missing or corrupted
- Log configuration errors and validation failures for debugging and monitoring
- Consider configuration backup and recovery mechanisms for critical applications

## Related AHK Concepts

- [IniWrite](./iniwrite.md) - Writing configuration values to INI files
- [FileRead](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) - Reading entire configuration files
- [FileExist](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) - Checking configuration file existence
- [RegRead](../../10_Language_Core/01-Functions/Built_In_Functions/regread.md) - Reading configuration from Windows registry
- [A_ScriptDir](../../00_Fundamentals/00-Variables_and_Expressions/builtin-variables.md) - Script-relative configuration paths

## Tags

#AutoHotkey #IniRead #Configuration #Settings #ConfigurationManagement #ApplicationSettings #UserPreferences #EnvironmentConfiguration #DataPersistence #SystemConfiguration
