# haskell-language-docker
[Hackage](https://hackage.haskell.org/package/language-docker)

- - -
Dockerfile parser, pretty-printer and embedded DSL

Provides de ability to parse docker files, a pretty-printer and EDSL for writting Dockerfiles in
Haskell.

- [Parsing files](#parsing-files)
- [Parsing strings](#parsing-strings)
- [Pretty-printing files](#pretty-printing-files)
- [Writing Dockerfiles in Haskell](#writing-dockerfiles-in-haskell)
- [Using the QuasiQuoter](#using-the-quasiquoter)
- [Templating Dockerfiles in Haskell](#templating-dockerfiles-in-haskell)
- [Using IO in the DSL](#using-io-in-the-dsl)

## Parsing files
```haskell
import Language.Docker
main = do
    ef <- parseFile "./Dockerfile"
    print ef
```

## Parsing strings
```haskell
import Language.Docker
main = do
    c <- readFile "./Dockerfile"
    print (parseString c)
```

## Pretty-printing files
```haskell
import Language.Docker
main = do
    Right d <- parseFile "./Dockerfile"
    putStr (prettyPrint d)
```

## Writing Dockerfiles in Haskell
```haskell
{-# LANGUAGE OverloadedStrings #-}
import Language.Docker
main = putStr $ toDockerfileStr $ do
    from "node"
    run "apt-get update"
    runArgs ["apt-get", "install", "something"]
    -- ...
```

## Using the QuasiQuoter
```haskell
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes       #-}
import Language.Docker
main = putStr $ toDockerfileStr $ do
    from "node"
    run "apt-get update"
    [edockerfile|
    RUN apt-get update
    CMD node something.js
    |]
    -- ...
```

## Templating Dockerfiles in Haskell
```haskell
{-# LANGUAGE FlexibleContexts  #-}
{-# LANGUAGE OverloadedStrings #-}
import Control.Monad
import Language.Docker
tags = ["7.8", "7.10", "8"]
cabalSandboxBuild packageName = do
    let cabalFile = packageName ++ ".cabal"
    run "cabal sandbox init"
    run "cabal update"
    add cabalFile ("/app/" ++ cabalFile)
    run "cabal install --only-dep -j"
    add "." "/app/"
    run "cabal build"
main =
    forM_ tags $ \tag -> do
        let df = toDockerfileStr $ do
            from ("haskell" `tagged` tag)
            cabalSandboxBuild "mypackage"
        writeFile ("./examples/templating-" ++ tag ++ ".dockerfile") df
```

## Using IO in the DSL
By default the DSL runs in the `Identity` monad. By running in IO we can
support more features like file globbing:

```haskell
{-# LANGUAGE OverloadedStrings #-}
import           Language.Docker
import qualified System.Directory     as Directory
import qualified System.FilePath      as FilePath
import qualified System.FilePath.Glob as Glob
main = do
    str <- toDockerfileStrIO $ do
        fs <- liftIO $ do
            cwd <- Directory.getCurrentDirectory
            fs <- Glob.glob "./test/*.hs"
            return (map (FilePath.makeRelative cwd) fs)
        from "ubuntu"
        mapM_ (\f -> add f ("/app/" ++ FilePath.takeFileName f)) fs
    putStr str
```

## License
GPLv3
