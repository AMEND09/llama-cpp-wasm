# llama-cpp-wasm

**WebAssembly** (Wasm) Build and Bindings for [llama.cpp](https://github.com/ggerganov/llama.cpp) and supported by [Tangled Group, Inc](https://tangledgroup.com).

![llama-cpp-wasm](docs/img/run-llama-cpp-in-browser-twitter-fs8.png)

## Online Demos

https://tangledgroup.github.io/llama-cpp-wasm/


## Build

```bash
git clone https://github.com/tangledgroup/llama-cpp-wasm.git
cd llama-cpp-wasm
./build-single-thread.sh
./build-multi-thread.sh
```

Once build is complete you can find `llama.cpp` built in `dist/llama-st` and `dist/llama-mt` directory.


## Deploy

Basically, you can copy/paste `dist/llama-st` or `dist/llama-mt` directory after build to your project and use as vanilla JavaScript library/module.

### Deploy to GitHub Pages ✅

To host the demo on GitHub Pages from this repository's `docs/` folder (recommended):

1. Build the project (creates the `docs/` assets):

   ```bash
   ./build-single-thread.sh
   ./build-multi-thread.sh
   ```

2. Commit and push the `docs/` folder to the `main` branch.

3. In the repository Settings → Pages, set the source to:
   - Branch: `main`
   - Folder: `/docs`

4. Save and wait a minute — your site will be available at `https://<your-username>.github.io/<repo-name>/`.

Notes:
- Links in the `docs/` pages are relative so they will work under the repo subpath.
- If you prefer automatic deployment, consider a GitHub Action that builds the site and pushes to the `gh-pages` branch.

Serve the root demo under `docs/`:

- This repository includes a `monitor.html` page that is a copy of the root `index.html`, adjusted to run from the `docs/` folder. When the Pages source is set to `main` branch `/docs` folder, open:

  `https://<your-username>.github.io/<repo-name>/monitor.html`

- If you'd rather use the root `index.html` as the site entry, set Pages source to the repository root (if GitHub Pages supports that for your repo), or move the root files into `docs/`.

Changelog (this repo):
- Fixed worker conflict that caused `invalid worker function to call: undefined` in browser workers by clearing Emscripten's global `onmessage` handler in the worker initializer. ✅
- Replaced absolute-root links in `docs/` HTML with relative links so the demo works under `https://<username>.github.io/<repo>/`.



**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <label for="prompt">Prompt:</label>
    <br/>

    <textarea id="prompt" name="prompt" rows="25" cols="80">Suppose Alice originally had 3 apples, then Bob gave Alice 7 apples, then Alice gave Cook 5 apples, and then Tim gave Alice 3x the amount of apples Alice had. How many apples does Alice have now? Let’s think step by step.</textarea>
    <br/>

    <label for="result">Result:</label>
    <br/>

    <textarea id="result" name="result" rows="25" cols="80"></textarea>
    <br/>
    
    <script type="module" src="example.js"></script>
  </body>
</html>
```


**example.js**

```javascript
// import { LlamaCpp } from "./llama-st/llama.js";
import { LlamaCpp } from "./llama-mt/llama.js";

const onModelLoaded = () => { 
  console.debug('model: loaded');
  const prompt = document.querySelector("#prompt").value;
  document.querySelector("#result").value = prompt;

  app.run({
    prompt: prompt,
    ctx_size: 4096,
    temp: 0.1,
    no_display_prompt: true,
  });
}

const onMessageChunk = (text) => {
  console.log(text);
  document.querySelector('#result').value += text;
};

const onComplete = () => {
  console.debug('model: completed');
};

const models = [
  'https://huggingface.co/Qwen/Qwen1.5-0.5B-Chat-GGUF/resolve/main/qwen2-beta-0_5b-chat-q8_0.gguf',
  'https://huggingface.co/Qwen/Qwen1.5-1.8B-Chat-GGUF/resolve/main/qwen1_5-1_8b-chat-q8_0.gguf',
  'https://huggingface.co/stabilityai/stablelm-2-zephyr-1_6b/resolve/main/stablelm-2-zephyr-1_6b-Q4_1.gguf',
  'https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf',
  'https://huggingface.co/TheBloke/phi-2-GGUF/resolve/main/phi-2.Q4_K_M.gguf'
];

const model = models[2]; // stablelm-2-zephyr-1_6b

const app = new LlamaCpp(
  model,
  onModelLoaded,          
  onMessageChunk,       
  onComplete,
);
```


## Run Example

First generate self-signed certificate.

```bash
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```

### Run Single Thread Example

```bash
npx http-server -S -C cert.pem
```

### Run Multi-threading Example

Copy `docs/server.js` to your working directory.

```bash
npm i express
node server.js
```

Then open in browser: https://127.0.0.1:8080/
