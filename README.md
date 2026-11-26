# Workshop: Building MCP Applications with Spring AI

## 1. Setup

### 1.1 Install multipass
#### 1.1.1 Ubuntu
On Ubuntu `multipass` may be installed as a snap:
```
sudo snap install multipass
```

#### 1.1.2 Debian
Install snapd using instructions [here](https://snapcraft.io/docs/tutorials/install-the-daemon/debian/#tutorials-install-the-daemon-debian).
```
sudo apt update
sudo apt install snapd
```
Then install the `multipass` snap:
```
sudo snap install multipass
```

#### 1.1.3 Fedora/RHEL
Please follow the instructions at [here](https://snapcraft.io/install/multipass/fedora) to enabled `snapd` and install multipass
as a snap:

```
sudo dnf install snapd
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install multipass
```
#### 1.1.4 Windows
Download [multipass-1.17.1+win-win64.msi](https://github.com/canonical/multipass/releases/download/v1.17.2/multipass-1.17.2+win-win64.msi) from the Multipass [releases page](https://github.com/canonical/multipass/releases/latest). 
Run the installer (administrator rights required). Open a new terminal afterwards, so that multipass is on your PATH.

#### 1.1.5 macOS
Download [multipass-1.17.1+mac-Darwin.pkg](https://github.com/canonical/multipass/releases/download/v1.17.2/multipass-1.17.2+mac-Darwin.pkg) from the Multipass [releases page](https://github.com/canonical/multipass/releases/latest) and run the installer package.

### 1.2 Install nodejs and npm
#### 1.1.1 Ubuntu
#### 1.1.2 Debian
#### 1.1.3 Fedora/RHEL
#### 1.1.4 Windows
#### 1.1.5 macOS

### 1.3 Preparing an Ubuntu 26.04 virtual machine
#### 1.3.1 Launch a multipass VM
```
multipass launch 26.04 --memory 4G --disk 30G --name mcp-demo
multipass shell mcp-demo
```

#### 1.3.2 Install OpenJDK 21
```
sudo apt update
sudo apt install openjdk-21-jdk-headless
```
#### 1.3.3 Install devpack-for-spring
```
sudo snap install devpack-for-spring --classic
```

#### 1.3.4 Install GLM 4.7
```
sudo snap install glm-4-7-flash
```
```
glm-4-7-flash status
```
The expected output is:
```
engine: cpu
services:
    server: active
    server-webui: active
entrypoints:
    openai:
        url: http://127.0.0.1:8354/v1
    webui:
        url: http://127.0.0.1:8355
model:
    name: glm-4.7-flash-30b-a3b
```
## Implementing an MCP server
```
devpack-for-spring boot start \
      --path $PWD/currency-converter-mcp \
      --project gradle-project \
      --language java \
      --boot-version 4.1.1 \
      --version 0.0.1-SNAPSHOT \
      --group com.example \
      --artifact demo \
      --name currency-converter-mcp \
      --description "A sample MCP server for currency conversions" \
      --package-name com.example.demo \
      --dependencies spring-ai-mcp-server,web \
      --packaging jar \
      --java-version 21
```

```
mkdir -p src/main/java/com/example/demo
```
code src/main/java/com/example/demo/CurrencyConverter.java
```
```
package com.example.demo;

import org.springframework.ai.mcp.annotation.McpTool;
import org.springframework.ai.mcp.annotation.McpToolParam;
import org.springframework.stereotype.Component;
import tools.jackson.databind.JsonNode;
import tools.jackson.databind.ObjectMapper;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

@Component
public class CurrencyConverter {

        private final HttpClient http = HttpClient.newHttpClient();
        private final ObjectMapper mapper = new ObjectMapper();

        @McpTool(description = "Convert an amount from one currency to another using today's exchange rate")
        public String convert(
                        @McpToolParam(description = "Source currency code, e.g. USD") String fromCurrency,
                        @McpToolParam(description = "Target currency code, e.g. EUR") String toCurrency,
                        @McpToolParam(description = "Amount to convert") double number) {
                if (number < 0) {
                        return "Error: amount must not be negative";
                }
                try {
                        var request = HttpRequest.newBuilder()
                                        .uri(URI.create("https://api.frankfurter.dev/v2/rate/" + fromCurrency + "/" + toCurrency))
                                        .GET().build();
                        var response = http.send(request, HttpResponse.BodyHandlers.ofString());
                        JsonNode json = mapper.readTree(response.body());
                        if (response.statusCode() != 200) {
                                return "Error: " + json.path("message").asString("HTTP " + response.statusCode());
                        }
                        double rate = json.get("rate").asDouble();
                        return "%s %s = %s %s (rate: %s)".formatted(number, fromCurrency, number * rate, toCurrency, rate);
                } catch (Exception e) {
                        return "Error: failed to fetch exchange rate: " + e.getMessage();
                }
        }

}
```
## Extending a chat-client for MCP
