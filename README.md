# Log4Xojo

Log4Xojo is a lightweight and thread-safe logging framework designed for Xojo applications. It provides robust logging capabilities, including support for multiple log levels, dynamic log file naming, file rotation, asynchronous file logging, and automatic protection against repeated file-write failures.

To learn more about this class, check out: https://blog.xojo.com/2024/11/26/log4xojo-a-more-powerful-way-to-manage-your-app-logging/.

---

## Features

- **Thread-Safe Logging**: Handles concurrent log messages across multiple threads.
- **Multiple Log Levels**: Supports `Debug`, `Info`, `Warning`, `Error`, and `Critical` levels.
- **Dynamic File Naming**: Includes the log name and current date in log file names for better organization.
- **File Rotation**: Automatically rotates log files based on size with configurable backup limits.
- **File Write Failure Protection**: Automatically disables file logging after repeated write failures to avoid runaway retry loops.
- **Fallback Failure Reporting**: When file logging is disabled, Log4Xojo reports the failure to `System.DebugLog` and `System.Log`.
- **Multiple Destinations**: Log messages can be sent to:
  - Debug console (`DebugLog`)
  - System logger (`SystemLog`)
  - Log files (`FileLog`)
  - All destinations (`All`)
- **Customizable File Path**: Allows specifying a base directory for log files.

---

## Usage Example

### Basic Example

```xojo
// Create a logger instance
Var appLog As New Log4Xojo("AppLog")

// Set the base location for log files
appLog.SetLogFilePath(SpecialFolder.Documents)

// Configure the logger
appLog.SetLogLevel(Log4Xojo.LogLevel.Info)
appLog.SetLogDestinations(Log4Xojo.LogDestination.FileLog)

// Log some messages
appLog.Log("Application started", Log4Xojo.LogLevel.Info)
appLog.Log("This is a warning", Log4Xojo.LogLevel.Warning)
appLog.Log("Critical error occurred", Log4Xojo.LogLevel.Critical, CurrentMethodName)

// Stop logging when done
appLog.StopLogging()
```

### Rotating Logs by Size

```xojo
// Set maximum log file size, e.g. 1 MB
appLog.SetMaxLogFileSize(1024 * 1024)

// Set the maximum number of backup files
appLog.SetMaxBackupFiles(5)
```

---

## File Write Failure Protection

Log4Xojo writes file logs through a background queue. If the log file cannot be written repeatedly, file logging is automatically disabled after a fixed number of consecutive failures.

This prevents the logger from repeatedly retrying a failed file write forever, which can otherwise create excessive CPU usage or trigger thread scheduler errors.

Common causes of repeated file-write failures include:

- The log folder no longer exists.
- The log folder is not writable.
- The log file is locked by another process.
- Antivirus or backup software temporarily blocks access.
- File rotation fails.
- The configured path points to an invalid location.
- The target log file path is occupied by a folder with the same name.

When the failure limit is reached, only file logging is disabled. Other destinations continue to work.

For example:

```xojo
appLog.SetLogDestinations(Log4Xojo.LogDestination.All)
```

If file logging fails repeatedly:

- `FileLog` stops writing.
- `DebugLog` continues.
- `SystemLog` continues.

This keeps the application logging useful diagnostic information without allowing the broken file destination to destabilize the app.

---

## MaxWriteFailures

`MaxWriteFailures` controls how many consecutive file-write failures are allowed before Log4Xojo disables file logging.

Default value:

```xojo
MaxWriteFailures = 5
```

This is a class constant. To change the threshold, edit the `MaxWriteFailures` constant in the Log4Xojo class.

Example behavior with the default value:

1. Log4Xojo attempts to write queued messages to the file.
2. The write fails.
3. The failed batch is requeued.
4. Log4Xojo waits briefly before retrying.
5. After 5 consecutive failures, file logging is disabled.
6. A critical diagnostic message is sent to `System.DebugLog` and `System.Log`.

The failure counter resets after a successful file write.

---

## Failure Reporting

When file logging disables itself, Log4Xojo reports the issue through fallback logging channels instead of trying to write the failure message to the same broken log file.

The fallback report includes:

- The number of consecutive write failures.
- The configured log file path.
- The last write error.
- The first failed log message from the failed batch.
- The failed batch size, if more than one message was being written.

Example fallback message:

```text
Log4Xojo: File logging disabled after 5 consecutive write failures. Path: C:\Logs\AppLog_2026-04-27.txt Last error: Access denied First failed log message: [2026-04-27 15:42:10] [ERROR] Database connection failed Failed batch size: 50
```

Because this fallback report may include the original log message, avoid logging sensitive data such as passwords, tokens, personal information, or full payment details.

---

## Stopping Logging

Call `StopLogging` before your application exits if you want Log4Xojo to finish processing queued file messages.

```xojo
appLog.StopLogging()
```

`StopLogging` lets the background logger drain the queue for a limited time. If the queue cannot be drained, pending file messages are discarded so shutdown can continue.

---

## Contributing

Contributions are welcome!

---

## License

This project is licensed under the MIT License. See the <a href="LICENSE">LICENSE</a> file for details.

---

## Acknowledgements

This class was inspired by popular logging frameworks such as <a href="https://logging.apache.org/log4j/">Log4J</a> and <a href="https://logging.apache.org/log4net/">Log4Net</a>, adapted for the Xojo ecosystem.
