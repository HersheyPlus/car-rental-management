# Zap Documentation

## Constant (zapcore)
### Level
* **DebugLevel** : logs are typically voluminous, and are usually disabled in production.
* **InfoLevel** : default logging priority.
* **WarnLevel** : more important than Info, but don't need individual human review.
* **ErrorLevel** : high-priority. If an application is running smoothly, it shouldn't generate any error-level logs.
* **DPanicLevel** : particularly important errors. In development the logger panics after writing the message.
* **PanicLevel** : logs a message, then panics.
* **FatalLevel** : logs a message, then calls os.Exit(1).

### Function
* ```func LevelFlag(name string, defaultLevel zapcore.Level, usage string) *zapcore.Level```: uses the standard library's flag.Var to declare a global flag with the specified name, default, and usage guidance. The returned value is a pointer to the value of the flag.
* ```func NewDevelopmentEncoderConfig() zapcore.EncoderConfig```: returns an opinionated EncoderConfig for development environments.
* ```func NewProductionEncoderConfig() zapcore.EncoderConfig```: returns an opinionated EncoderConfig for production environments.
* ```func NewStdLog(l *Logger) *log.Logger```: returns a *log.Logger which writes to the supplied zap Logger at InfoLevel. To redirect the standard library's package-global logging functions, use ```RedirectStdLog``` instead.
* ```func NewStdLogAt(l *Logger, level zapcore.Level) (*log.Logger, error)```: returns *log.Logger which writes to supplied zap logger at required level.
* ```func RedirectStdLog(l *Logger) func()```: redirects output from the standard library's package-global logger to the supplied logger at InfoLevel.
* ```func RedirectStdLogAt(l *Logger, level zapcore.Level) (func(), error)```: redirects output from the standard library's package-global logger to the supplied logger at the specified level.
* ```func RegisterEncoder(name string, constructor func(zapcore.EncoderConfig) (zapcore.Encoder, error)) error```: registers an encoder constructor, which the Config struct can then reference. By default, the "json" and "console" encoders are registered.
* ```func RegisterSink(scheme string, factory func(*url.URL) (Sink, error)) error```: registers a user-supplied factory for all sinks with a particular scheme.
* ```func ReplaceGlobals(logger *Logger) func()```: replaces the global Logger and SugaredLogger, and returns a function to restore the original values. It's safe for concurrent use.

## AtomicLevel
* An ```AtomicLevel``` is an atomically changeable, dynamic logging level. It lets you safely change the log level of a tree of loggers (the root logger and any children created by adding context) at runtime.
* The ```AtomicLevel``` itself is an http.Handler that serves a JSON endpoint to alter its level.
* ```func NewAtomicLevel() AtomicLevel```:  creates an AtomicLevel with InfoLevel and above logging enabled.
* ```func NewAtomicLevelAt(l zapcore.Level) AtomicLevel```: It is a convenience function that creates an AtomicLevel and then calls SetLevel with the given level.
* ```func ParseAtomicLevel(text string) (AtomicLevel, error)```: parses an AtomicLevel based on a lowercase or all-caps ASCII representation of the log level. If the provided ASCII representation is invalid an error is returned. This is particularly useful when dealing with text input to configure log levels.
* ```func (lvl AtomicLevel) Enabled(l zapcore.Level) bool```: implements the zapcore.LevelEnabler interface, which allows the AtomicLevel to be used in place of traditional static levels.
* ```func (lvl AtomicLevel) Level() zapcore.Level```: returns the minimum enabled log level.
* ```func (lvl AtomicLevel) MarshalText() (text []byte, err error)```: marshals the ```AtomicLevel``` to a byte slice. It uses the same text representation as the static zapcore.Levels ("debug", "info", "warn", "error", "dpanic", "panic", and "fatal").
* ```func (lvl AtomicLevel) ServeHTTP(w http.ResponseWriter, r *http.Request)```: alters the logging level.
* ```func (lvl AtomicLevel) SetLevel(l zapcore.Level)```: 
* ```func (lvl AtomicLevel) String() string```: returns the string representation of the underlying Level.
* ```func (lvl *AtomicLevel) UnmarshalText(text []byte) error```: unmarshals the text to an ```AtomicLevel```. It uses the same text representations as the static zapcore.Levels ("debug", "info", "warn", "error", "dpanic", "panic", and "fatal").

## Config
* offers a declarative way to construct a logger. It doesn't do anything that can't be done with New, Options, and the various zapcore.WriteSyncer and zapcore.Core wrappers, but it's a simpler way to toggle common options.
* ```func NewDevelopmentConfig() Config```: builds a reasonable default development logging configuration. Logging is enabled at DebugLevel and above, and uses a console encoder. Logs are written to standard error. Stacktraces are included on logs of WarnLevel and above. DPanicLevel logs will panic.
* ```func NewProductionConfig() Config```: builds a reasonable default production logging configuration. Logging is enabled at InfoLevel and above, and uses a JSON encoder. Logs are written to standard error. Stacktraces are included on logs of ErrorLevel and above. DPanicLevel logs will not panic, but will write a stacktrace.
* ```func (cfg Config) Build(opts ...Option) (*Logger, error)```: constructs a logger from the Config and Options.



## Fields
* ```func Array(key string, val zapcore.ArrayMarshaler)```
* ```func Bool(key string, val bool)```
* ```func Dict(key string, val ...Field)```
* ```func Duration(key string, val time.Duration)```
* ```func Error(err error)```
* ```func Float32(key string, val float32)```
* ```func Float64(key string, val float64)```
* ```func Int(key string, val int)```
* ```func NamedError(key string, err error)```
* ```func Namespace(key string)```
* ```func Skip()```: useful when handling invalid inputs in other Field constructors.
* ```func Stack(key string) Field```
* ```func StackSkip(key string, skip int) Field```
* ```func String(key string, val string)```
* ```func Time(key string, val time.Time)```
* ```func Uint(key string, val uint)```

## LevelEnablerFunc
* ```type LevelEnablerFunc func(zapcore.Level) bool```: It is a convenient way to implement zapcore.LevelEnabler with an anonymous function, useful when splitting log output between different outputs.
* ```func (f LevelEnablerFunc) Enabled(lvl zapcore.Level) bool``` : calls the wrapped function
## Logger
* ```func L() *Logger```: returns the global Logger
* ```func Must(logger *Logger, err error) *Logger```: Must is a helper that wraps a call to a function returning (*Logger, error) and panics if the error is non-nil. It is intended for use in variable initialization.
### New
* ```func New(core zapcore.Core, options ...Option) *Logger```: New constructs a new Logger from the provided zapcore.Core and Options. If the passed zapcore. Core is nil, it falls back to using a no-op implementation. This is the most flexible way to construct a Logger, but also the most verbose.
* ```func NewDevelopment(options ...Option) (*Logger, error)```: builds a development Logger that writes DebugLevel and above logs to standard error in a human-friendly format It's a shortcut for ```NewDevelopmentConfig().Build(...Option)```
* ```func NewExample(options ...Option) *Logger```: NewExample builds a Logger that's designed for use in zap's testable examples. It writes DebugLevel and above logs to standard out as JSON, but omits the timestamp and calling function to keep example output short and deterministic.
* ```func NewProduction(options ...Option) (*Logger, error)```:NewProduction builds a sensible production Logger that writes InfoLevel and above logs to standard error as JSON. It's a shortcut for ```NewProductionConfig().Build(...Option)```.
### Logger Functions
* ```func (log *Logger) Check(lvl zapcore.Level, msg string) *zapcore.CheckedEntry```: Check returns a CheckedEntry if logging a message at the specified level is enabled. It's a completely optional optimization.
* ```func (log *Logger) Core() zapcore.Core```: returns the Logger's underlying zapcore.Core.
* ```func (log *Logger) DPanic(msg string, fields ...Field)```:  logs a message at DPanicLevel.
* ```func (log *Logger) Panic(msg string, fields ...Field)``` :  logs a message at PanicLevel.
* ```func (log *Logger) Debug(msg string, fields ...Field)```:  logs a message at DebugLevel.
* ```func (log *Logger) Error(msg string, fields ...Field)```: logs a message at ErrorLevel.
* ```func (log *Logger) Fatal(msg string, fields ...Field)```: logs a message at FatalLevel.
* ```func (log *Logger) Info(msg string, fields ...Field)```: logs a message at InfoLevel.
* ```func (log *Logger) Level() zapcore.Level```: reports the minimum enabled level for this logger.
* ```func (log *Logger) Log(lvl zapcore.Level, msg string, fields ...Field)```: logs a message at the specified level.
* ```func (log *Logger) Name() string```: returns the Logger's underlying name, or an empty string if the logger is unname.
* ```func (log *Logger) Named(s string) *Logger```: adds a new path segment to the logger's name.
* ```func (log *Logger) Sugar() *SugaredLogger```: Sugar wraps the Logger to provide a more ergonomic, but slightly slower, API. Sugaring a Logger is quite inexpensive, so it's reasonable for a single application to use both Loggers and SugaredLoggers, converting between them on the boundaries of performance-sensitive code.
* ```func (log *Logger) Sync() error```: Sync calls the underlying Core's Sync method, flushing any buffered log entries.
* ```func (log *Logger) Warn(msg string, fields ...Field)```:  logs a message at WarnLevel.
* ```func (log *Logger) With(fields ...Field) *Logger```:  creates a child logger and adds structured context to it. Fields added to the child don't affect the parent.
* ```func (log *Logger) WithLazy(fields ...Field) *Logger```: creates a child logger and adds structured context to it lazily.
* ```func (log *Logger) WithOptions(opts ...Option) *Logger```: clones the current Logger, applies the supplied Options, and returns the resulting Logger.
## Option
* ```func AddCaller() Option```
* ```func AddCallerSkip(skip int) Option```
* ```func AddStacktrace(lvl zapcore.LevelEnabler) Option```
* ```func Development() Option```
* ```func ErrorOutput(w zapcore.WriteSyncer) Option```
* ```func Fields(fs ...Field) Option```
* ```func Hooks(hooks ...func(zapcore.Entry) error) Option```
* ```func IncreaseLevel(lvl zapcore.LevelEnabler) Option```
* ```func WithCaller(enabled bool) Option```
* ```func WithClock(clock zapcore.Clock) Option```
* ```func WithFatalHook(hook zapcore.CheckWriteHook) Option```
* ```func WithPanicHook(hook zapcore.CheckWriteHook) Option```
* ```func WrapCore(f func(zapcore.Core) zapcore.Core) Option```
## Sink
* defines the interface to write to and close logger destinations.
```
type Sink interface {
	zapcore.WriteSyncer
	io.Closer
}
```