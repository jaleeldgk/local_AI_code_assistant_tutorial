# General Local AI Usage

Requirements: Apple Silicon with 32GB or more RAM

1. **Download LMStudio for MacOS :**
   - [LMStudio](https://lmstudio.ai/)

2. **In LMStudio, search for:**
   - `bartowski/Codestral-22B-v0.1-GGUF`

3. **Download the model that ends with `Q6_K` on the right:**
   - **Note:** The `Q6_K` model consumes 18.40GB of RAM when loaded. If you have more RAM to spare, you may try the `Q8_0` model which requires a minimum of 23.30GB RAM and has slightly higher quality and lower speed. The bigger model will require unlocking the wired memory limit with sysctl:
`sudo sysctl iogpu.wired_limit_mb=50000`

4. **In the AI Chat window, load the Codestral model.**

5. **Ensure the following settings on the right (expand Advanced Configuration)**
   - **Preset:** Mistral Instruct
   - **Clear the System Prompt**
   - **Context Length:** 8192
   - **Temperature:** 0.8
   - **Tokens to generate:** -1
   - **CPU Threads:** 10
   - **GPU Offload:** Max
   - **GPU Backend:** Metal llama.cpp
   - **(further below) Flash Attention:** on

Example uses for AI chat:
 - Chat about code, programming concepts
 - Ask it to generate terminal commands, Docker commands, K8s etc.
 - Ask it to modify formatting like JSON, add markdown, etc.
 - Ask it to explain foreign code
 - Ask it to rework code in a certain way, write comments, PR description, unit tests
 - Paste in a PR diff to review (can be private code as it runs fully local)
 - Paste in a GitHub issue discussion (or any long text) and ask it to summarize
 - Modify system prompt (tab on the right) to force a certain kind of output (e.g. "You summarize discussions")

Test the AI directly in LMStudio chat before proceeding further. If you see errors such as [control_29] in the output window, it could mean the LLM did not fit into RAM. You might need to turn off some other apps or go for a smaller Q# model like Q4. Ignore the iQ# versions.

Next we will configure the VSCode plugin which connects to LMStudio. VSCode can be in a VM if you prefer, with LMStudio on host. It needs to be able to connect to the LMStudio server running on the host.

# VSCode Local AI Plugin (Continue.dev)

1. **Set up LMStudio first as in point 1.**

2. **Enable API server in LMStudio:**
   - Select the arrow tab (Local Server) on the left.
   - Click "Start Server".

3. **Install Continue.dev plugin for VSCode (also available in Jetbrains):**
   - Go to `View` -> `Extensions`.
   - Search for "Continue".
   - Install the plugin.

4. **Continue.dev configuration:**
   - Before doing anything, go to Continue plugin settings and disable Telemetry on the main page.
   - Open the config file:
     ```sh
     vim ~/.continue/config.json
     ```
   - Replace it with:
     ```json
     {
       "models": [
         {
           "title": "LM Studio",
           "provider": "lmstudio",
           "model": "codestral",
           "apiBase": "http://192.168.0.137:1234/v1/"
         },
         {
           "title": "Llama 3",
           "provider": "ollama",
           "model": "llama3"
         },
         {
           "title": "Ollama",
           "provider": "ollama",
           "model": "AUTODETECT"
         }
       ],
       "customCommands": [
         {
           "name": "test",
           "prompt": "{{{ input }}}\n\nWrite a comprehensive set of unit tests for the selected code. It should setup, run tests that check for correctness including important edge cases, and teardown. Ensure that the tests are complete and sophisticated. Give the tests just as chat output, don't edit any file.",
           "description": "Write unit tests for highlighted code"
         }
       ],
       "tabAutocompleteModel": {
         "title": "codestral",
         "provider": "lmstudio",
         "model": "codestral",
         "apiBase": "http://192.168.0.137:1234/v1/"
       },
       "allowAnonymousTelemetry": false,
       "embeddingsProvider": {
         "provider": "transformers.js"
       }
     }
     ```
   - **Note:** Replace BOTH `apiBase` URLs with the IP of your MacOS host where LMStudio is running (check with `ifconfig -a | grep inet`).

- **Restart VSCode.**
- **Test connection:**
  - Open the Continue tab in VSCode (on the left) and type in 'hello'. You may use this window as a normal chatbot AI running fully locally.
  - You can select any code and add it to chat (Ctrl+L) or Edit (Ctrl+I).
  - Press Ctrl+I in a code window, and a window will pop up to write a prompt for generating any code. You can then accept or reject it.
  - Verify that tab auto-complete works (AI suggests code continuation in grey, press TAB to accept).
  - You may try a partially written factorial function in the language of your choice to test TAB code completion
  - If auto-complete is stuck, you can always Ctrl+I and directly prompt it to continue the code.
  - You can always disable tab auto-complete in Continue extension settings in VSCode
  - You can expect text generation performance of about 1 word per second 

- **Learn how to provide additional context here:**
  - [Context Providers](https://docs.continue.dev/customization/context-providers). You can also click "Add Context" in the chat window with options to bring in files, folders, terminal output into prompt context.

- **Try codebase awareness - click 'Use codebase' in the Continue prompt window when writing the prompt. Continue will vectorize your codebase so it can intelligently bring in snippets of code that are releant to the prompt. You can ask 'where is X done' in the codebase and it will find it.**

- **Learn how to use Continue commands directly in the code window:**
  - [Slash Commands](https://docs.continue.dev/customization/slash-commands).

# Advanced notes

You can also try other LLM models like:
- `TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF` - use `Q4_K_M` version
- `TheBloke/deepseek-coder-33B-instruct-GGUF` - use `Q4_K_M` version and change Prompt Format (right tab) to Deepseek Coder

You may need to increase context length (both for AI Chat window and Local Server) from 8192 to a higher value when pasting long discussions or a lot of code. Codestral and Mixtral support up to 32768 tokens.

High amount of context increases RAM use so above a certain amount of text you can get errors. Use smaller Q size if you want to use a lot of context.

Use system prompt (right tab) if the model is not behaving the way you want to. Keep the system prompt simple. Remember to change it or clear it when you move to a different topic.
