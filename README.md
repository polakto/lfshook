# Local Filesystem Hook for Logrus

[![GoDoc](https://godoc.org/github.com/rifflock/lfshook?status.svg)](http://godoc.org/github.com/rifflock/lfshook)

Sometimes developers like to write directly to a file on the filesystem. This is a hook for [`logrus`](https://github.com/sirupsen/logrus) which designed to allow users to do that. The log levels are dynamic at instantiation of the hook, so it is capable of logging at some or all levels.

## Example
```go
import (
	"github.com/rifflock/lfshook"
	"github.com/sirupsen/logrus"
)

var Log *logrus.Logger

func NewLogger() *logrus.Logger {
	if Log != nil {
		return Log
	}

	pathMap := lfshook.PathMap{
		logrus.InfoLevel:  "/var/log/info.log",
		logrus.ErrorLevel: "/var/log/error.log",
	}

	Log = logrus.New()
	Log.Hooks.Add(lfshook.NewHook(
		pathMap,
		&logrus.JSONFormatter{},
	))
	return Log
}
```

### Formatters
`lfshook` will strip colors from any `TextFormatter` type formatters when writing to local file, because the color codes don't look great in file.

If no formatter is provided via `lfshook.NewHook`, a default text formatter will be used.

### Log rotation
In order to enable automatic log rotation it's possible to provide an io.Writer instead of the path string of a log file.
In combination with packages like [file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs) log rotation can easily be achieved.

```go
package main

import (
	rotatelogs "github.com/lestrrat-go/file-rotatelogs"
	"github.com/rifflock/lfshook"
	"github.com/sirupsen/logrus"
)

var Log *logrus.Logger

func NewLogger() *logrus.Logger {
	if Log != nil {
		return Log
	}

	path := "/var/log/go.log"
	writer := rotatelogs.New(
		path+".%Y%m%d%H%M",
		rotatelogs.WithLinkName(path),
		rotatelogs.WithMaxAge(time.Duration(86400)*time.Second),
		rotatelogs.WithRotationTime(time.Duration(604800)*time.Second),
	)

	logrus.Hooks.Add(lfshook.NewHook(
		lfshook.WriterMap{
			logrus.InfoLevel:  writer,
			logrus.ErrorLevel: writer,
		},
		&logrus.JSONFormatter,
	))

	Log = logrus.New()
	Log.Hooks.Add(lfshook.NewHook(
		pathMap,
		&logrus.JSONFormatter{},
	))

	return Log
}
```

### Note:
User who run the go application must have read/write permissions to the selected log files. If the files do not exists yet, then user must have permission to the target directory.

### Fork note:
This fork is able to filter incoming entries by "category" atttribute of entry. By default, all entries are suported. When at least one category for this hook is set up, then every incoming entry is checked for "category" attribute and its value checked if supported.
#### Usage
When we want to log all levels but only with "category" attribute set to "statistics", use this code.

```go
package main

import (
	"github.com/polakto/lfshook"
	"github.com/sirupsen/logrus"
)

var Log *logrus.Logger

// NewLoger terurns new instance of logger
func NewLogger() *logrus.Logger {
	if Log != nil {
		return Log
	}

	Log = logrus.New()

	pathMap := lfshook.PathMap{
		logrus.InfoLevel:  "./log/statistics.log",
		logrus.DebugLevel: "./log/statistics.log",
		logrus.WarnLevel:  "./log/statistics.log",
		logrus.ErrorLevel: "./log/statistics.log",
		logrus.PanicLevel: "./log/statistics.log",
		logrus.FatalLevel: "./log/statistics.log",
	}
	senderSentHook := lfshook.NewHook(
		pathMap,
		&logrus.JSONFormatter{},
	)
	senderSentHook.AddCategory("statistics")
	Log.Hooks.Add(senderSentHook)
	return Log
}
```