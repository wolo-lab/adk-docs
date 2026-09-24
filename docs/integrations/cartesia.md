---
catalog_title: Cartesia
catalog_description: Generate speech, localize voices, and create audio content
catalog_icon: /integrations/assets/cartesia.png
catalog_tags: ["mcp"]
---

# Cartesia MCP tool for ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span>
</div>

The [Cartesia MCP Server](https://github.com/cartesia-ai/cartesia-mcp) connects
your ADK agent to the [Cartesia](https://cartesia.ai/) AI audio platform. This
integration gives your agent the ability to generate speech, localize voices
across languages, and create audio content using natural language.

## Use cases

- **Text-to-Speech Generation**: Convert text into natural-sounding speech
  using Cartesia's diverse voice library, with control over voice selection and
  output format.

- **Voice Localization**: Transform existing voices into different languages
  while preserving the original speaker's characteristics—ideal for
  multilingual content creation.

- **Audio Infill**: Fill gaps between audio segments to create smooth
  transitions, useful for podcast editing or audiobook production.

- **Voice Transformation**: Convert audio clips to sound like different voices
  from Cartesia's library.

## Prerequisites

- Sign up for a [Cartesia account](https://play.cartesia.ai/sign-in)
- Generate an [API key](https://play.cartesia.ai/keys) from the Cartesia
  playground

## Use with agent

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        CARTESIA_API_KEY = "YOUR_CARTESIA_API_KEY"

        root_agent = Agent(
            model="gemini-flash-latest",
            name="cartesia_agent",
            instruction="Help users generate speech and work with audio content",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="uvx",
                            args=["cartesia-mcp"],
                            env={
                                "CARTESIA_API_KEY": CARTESIA_API_KEY,
                                # "OUTPUT_DIRECTORY": "/path/to/output",  # Optional
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Local MCP Server"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const CARTESIA_API_KEY = "YOUR_CARTESIA_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-flash-latest",
            name: "cartesia_agent",
            instruction: "Help users generate speech and work with audio content",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "uvx",
                        args: ["cartesia-mcp"],
                        env: {
                            CARTESIA_API_KEY: CARTESIA_API_KEY,
                            // OUTPUT_DIRECTORY: "/path/to/output",  // Optional
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

=== "Go"

    === "Local MCP Server"

        ```go
        package main

        import (
        	"context"
        	"log"
        	"os"
        	"os/exec"

        	"github.com/modelcontextprotocol/go-sdk/mcp"
        	"google.golang.org/genai"

        	"google.golang.org/adk/v2/agent"
        	"google.golang.org/adk/v2/agent/llmagent"
        	"google.golang.org/adk/v2/cmd/launcher"
        	"google.golang.org/adk/v2/cmd/launcher/full"
        	"google.golang.org/adk/v2/model/gemini"
        	"google.golang.org/adk/v2/tool"
        	"google.golang.org/adk/v2/tool/mcptoolset"
        )

        const cartesiaAPIKey = "YOUR_CARTESIA_API_KEY"

        func main() {
        	ctx := context.Background()

        	model, err := gemini.NewModel(ctx, "gemini-flash-latest", &genai.ClientConfig{
        		APIKey: os.Getenv("GOOGLE_API_KEY"),
        	})
        	if err != nil {
        		log.Fatalf("Failed to create the model: %v", err)
        	}

        	server := exec.CommandContext(ctx, "uvx", "cartesia-mcp")
        	// Forward only what uvx needs, plus the Cartesia key. The parent environment
        	// may hold unrelated secrets, such as the GOOGLE_API_KEY read above.
        	server.Env = []string{
        		"CARTESIA_API_KEY=" + cartesiaAPIKey,
        		// "OUTPUT_DIRECTORY=/path/to/output", // Optional
        	}
        	for _, k := range []string{
        		"PATH", "HOME", // POSIX
        		"XDG_CACHE_HOME", "XDG_DATA_HOME", "UV_CACHE_DIR", // uv cache and tool dirs
        		"APPDATA", "LOCALAPPDATA", "TEMP", "USERPROFILE", // Windows
        	} {
        		if v, ok := os.LookupEnv(k); ok {
        			server.Env = append(server.Env, k+"="+v)
        		}
        	}

        	cartesia, err := mcptoolset.New(mcptoolset.Config{
        		Transport: &mcp.CommandTransport{Command: server},
        	})
        	if err != nil {
        		log.Fatalf("Failed to create the Cartesia tool set: %v", err)
        	}

        	rootAgent, err := llmagent.New(llmagent.Config{
        		Model:       model,
        		Name:        "cartesia_agent",
        		Instruction: "Help users generate speech and work with audio content",
        		Toolsets:    []tool.Toolset{cartesia},
        	})
        	if err != nil {
        		log.Fatalf("Failed to create the agent: %v", err)
        	}

        	l := full.NewLauncher()
        	cfg := &launcher.Config{AgentLoader: agent.NewSingleLoader(rootAgent)}
        	if err := l.Execute(ctx, cfg, os.Args[1:]); err != nil {
        		log.Fatalf("Run failed: %v\n\n%s", err, l.CommandLineSyntax())
        	}
        }
        ```

## Available tools

Tool | Description
---- | -----------
`text_to_speech` | Convert text to audio using a specified voice
`list_voices` | List all available Cartesia voices
`get_voice` | Get details about a specific voice
`clone_voice` | Clone a voice from audio samples
`update_voice` | Update an existing voice
`delete_voice` | Delete a voice from your library
`localize_voice` | Transform a voice into a different language
`voice_change` | Convert an audio file to use a different voice
`infill` | Fill gaps between audio segments

## Configuration

The Cartesia MCP server can be configured using environment variables:

Variable | Description | Required
-------- | ----------- | --------
`CARTESIA_API_KEY` | Your Cartesia API key | Yes
`OUTPUT_DIRECTORY` | Directory to store generated audio files | No

## Additional resources

- [Cartesia MCP Server Repository](https://github.com/cartesia-ai/cartesia-mcp)
- [Cartesia MCP Documentation](https://docs.cartesia.ai/integrations/mcp)
- [Cartesia Playground](https://play.cartesia.ai/)
