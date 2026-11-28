# Workshop: Building MCP Applications with Spring AI

## 1 Setup (30m)
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
multipass launch 26.04 --cpus 4 --memory 10G --disk 36G --name mcp-demo
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

#### 1.3.4 Install Qwen3
Install the Qwen3 inference snap. The installation may take a few minutes.
```
sudo snap install qwen3
```
The qwen3 endpoint should be available immediately after install. Check with this command:
```
qwen3 status
```

The expected output is:
```
engine: cpu
services:
    server: active
    server-webui: active
entrypoints:
    openai:
        url: http://127.0.0.1:8338/v1
    webui:
        url: http://127.0.0.1:8339
model:
    name: qwen3-8b
```

Also, ensure that the model is loaded by issuing:
```
curl -s http://127.0.0.1:8338/v1/models
```
This should not report an `unavailable_error`.

## 2 Implementing an MCP server (20m)

### 2.1 Initialize the project
Initialize a Spring Boot project using `devpack-for-spring`:
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
Note the dependencies used in the above command.


### 2.2 Implementation

#### 2.2.1 Implementing a CurrencyConverter using the frankfurter API 

Add the following code to `src/main/java/com/example/demo/CurrencyConverter.java`:
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

#### 2.2.2 Adding configuration properties

Add the following properties to `src/main/resources/application.properties`:
```
server.port=9090
spring.ai.mcp.server.protocol=STREAMABLE
```

### 2.3 Testing the MCP

#### 2.3.1 Run the application
Run the application using gradle:
```
./gradlew bootRun
```
#### 2.3.2 Note the IP address of the VM
In a new terminal window, issue the following command:
```
multipass info mcp-demo | grep "IP"
```
Note the IPv4 address echo'd above.

#### 2.3.2 Test using MCP Inspector
In the same window, issue the following command:
```
npx @modelcontextprotocol/inspector
```

This should open up a browser tab with the MCP Inspector UI. Follow these steps to run a test:
 - Click **Add Servers** -> **Add manually**
 - In the Add server window:
     - set `Server ID` to `currency-conversion-mcp`
     - set `Transport` to `streamable-http` and
     - set `URL` to `http://[ip-address]:9090/mcp` (use IP address from the previous step)
- Click **Add**. The Servers tab will show a new UI element for this MCP server.
  <img width="594" height="332" alt="Screenshot From 2026-11-28 12-20-18" src="https://github.com/user-attachments/assets/fe5f6328-57fb-4b88-9127-fd295338cc64" />

- Click the toggle button at the top-right of this UI element, to connect to the MCP server. A successful connection (or a failure) should be indicated by the user-interface.
  <img width="682" height="331" alt="Screenshot From 2026-11-28 12-21-05" src="https://github.com/user-attachments/assets/70dbef52-bf38-4614-a5a7-f397432c57ae" />

- If the connection is successful, click on the Tools tab. In the tools view, you should see the convert tool listed.
  <img width="700" height="359" alt="Screenshot From 2026-11-28 12-22-52" src="https://github.com/user-attachments/assets/b289dd4a-24fb-4f72-affe-20d76d0c84ff" />

- Click on convert. This should open up a dialog listing the tool-parameters.
  <img width="1008" height="554" alt="Screenshot From 2026-11-28 12-23-48" src="https://github.com/user-attachments/assets/16fcf26b-0db6-45c0-9cbd-3941f9aea611" />

- Entering valid values and clicking Execute Tool should show this:
  <img width="1005" height="194" alt="Screenshot From 2026-11-28 12-24-28" src="https://github.com/user-attachments/assets/1be178a5-3501-4794-bf57-45c18bcf06f5" />


The MCP server is now ready and we can move to the client implementation.

## 3 Extending a Spring AI-based chat-client with MCP (30m)

### 3.1 Testing an existing chat-client application without MCP

### 3.2 Extending the chat-client to include MCP

### 3.3 Testing the chat-client with MCP
