# wai-logging-effect-google
> Create wai log entry with google logging format.

Middleware for [wai](https://www.stackage.org/lts-7.4/package/wai-3.2.1.1) which create log entry as 
[Google Cloud Platform's](https://cloud.google.com/) [Logging](https://cloud.google.com/logging/docs/) service format.

## Installation

This package is not in hackage yet. So for install it add to `stack.yaml`
in `packages` next:
```
- location:
  git: https://github.com/d61h6k4/wai-logging-effect-google.git
  commit: <insert commit here>
```
and add `wai-logging-effect-google` to your `cabal` file.

## Usage example

Here example how I use it with [wai](https://hackage.haskell.org/package/wai) server on GCP. Additionally here used [logging-effect-google](https://github.com/d61h6k4/logging-effect-google.git). Here google logging handler used to store logs from wai.

```haskell
app :: Application
app request respond =
    case rawPathInfo request of
        "/events" ->
            events request >>= respond

main :: IO ()
main = do
    lgr <- newLogger Debug stdout
    env <- newEnv <&> (envScopes .~ ( loggingWriteScope ! pubSubScope)) . (envLogger .~ lgr)
    withGoogleLoggingHandler
        (defaultBatchingOptions
        { flushMaxDelay = 10 * 1000000
        , flushMaxQueueSize = 128
        })
        env
        (Just
            ("projects/" <> ("test_project_id") <>
              ("/logs/negotians")))
        (Just
            ((monitoredResource & mrLabels .~
              (Just
                    (monitoredResourceLabels
                        (HashMap.fromList
                              ([ ( "instanceId"
                                , ("0000"))
                              , ("zone", ("eu-west1-c"))]))))) &
              mrType .~
              (Just "gce_instance")))
        Nothing $
    \wailogger ->
        run 3000 $ loggerMiddleware wailogger $ gzip def $
        app 
```

## Development setup

Fork it.

## Release History

## Meta

Distributed under BSD license. See ``LICENSE`` for more information.
