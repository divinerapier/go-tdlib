# go-tdlib

Go wrapper for [TDLib (Telegram Database Library)](https://github.com/tdlib/td) with full support of TDLib v1.2.0

## TDLib installation

### Ubuntu 18.04 / Debian 9

```bash
apt-get update -y
apt-get install -y \
    build-essential \
    ca-certificates \
    ccache \
    cmake \
    git \
    gperf \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev
git clone --depth 1 -b "v1.2.0" "https://github.com/tdlib/td.git" ./tdlib-src
mkdir ./tdlib-src/build
cd ./tdlib-src/build
cmake -j$(getconf _NPROCESSORS_ONLN) -DCMAKE_BUILD_TYPE=Release ..
cmake -j$(getconf _NPROCESSORS_ONLN) --build .
make -j$(getconf _NPROCESSORS_ONLN) install
rm -rf ./../../tdlib-src
```


## Usage

### Client

[Register an application](https://my.telegram.org/apps) to obtain an api_id and api_hash 

```go
package main

import (
    "log"

    "github.com/zelenin/go-tdlib/client"
)

func main() {
    client.SetLogVerbosityLevel(1)
    client.SetLogFilePath("/dev/stderr")
    
    // client authorizer
    authorizer := client.ClientAuthorizer()
    go client.CliInteractor(authorizer)
    
    // or bot authorizer
    botToken := "000000000:gsVCGG5YbikxYHC7bP5vRvmBqJ7Xz6vG6td"
    authorizer := client.BotAuthorizer(botToken)
    
    const (
        apiId   = 00000
        apiHash = "8pu9yg32qkuukj83ozaqo5zzjwhkxhnk"
    )

    authorizer.TdlibParameters <- &client.TdlibParameters{
        UseTestDc:              false,
        DatabaseDirectory:      "./.tdlib/database",
        FilesDirectory:         "./.tdlib/files",
        UseFileDatabase:        true,
        UseChatInfoDatabase:    true,
        UseMessageDatabase:     true,
        UseSecretChats:         false,
        ApiId:                  apiId,
        ApiHash:                apiHash,
        SystemLanguageCode:     "en",
        DeviceModel:            "Server",
        SystemVersion:          "1.0.0",
        ApplicationVersion:     "1.0.0",
        EnableStorageOptimizer: true,
        IgnoreFileNames:        false,
    }

    tdlibClient, err := client.NewClient(authorizer)
    if err != nil {
        log.Fatalf("NewClient error: %s", err)
    }

    me, err := tdlibClient.GetMe()
    if err != nil {
        log.Fatalf("GetMe error: %s", err)
    }

    log.Printf("Me: %s %s [%s]", me.FirstName, me.LastName, me.Username)
}

```

### Receive updates

```go
responses := make(chan client.Type, 100)
tdlibClient, err := client.NewClient(authorizer, client.WithListener(responses))
if err != nil {
    log.Fatalf("NewClient error: %s", err)
}

for response := range responses {
    if response.GetClass() == client.ClassUpdate {
        log.Printf("%#v", response)
    }
}
```

## Notes

* WIP. Library API can be changed in the future
* The package includes a .tl-parser and generated [json-schema](https://github.com/zelenin/go-tdlib/tree/master/data) for creating libraries in other languages

## Author

[Aleksandr Zelenin](https://github.com/zelenin/), e-mail: [aleksandr@zelenin.me](mailto:aleksandr@zelenin.me)
