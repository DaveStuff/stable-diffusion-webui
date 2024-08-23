The `provisioning_download` function is a Bash function designed to handle downloading files from specific URLs that may require authentication. It identifies whether the URL is from Hugging Face or Civitai, determines the type of authentication needed, and then uses `wget` to download the file. Here's a detailed breakdown of how the function works:

### Function Breakdown:

1. **Hugging Face URL Handling:**

    ```bash
    if [[ -n $HF_TOKEN && $1 =~ ^https:\/\/huggingface\.co\/.*\.(safetensors|bin|ckpt|onnx|pt|pkl|yaml|yml|zip)$ ]]; then
        auth_token="$HF_TOKEN"
        url_type=hf
    ```

    - **`-n $HF_TOKEN`**: Checks if the `HF_TOKEN` environment variable (Hugging Face authentication token) is set and not empty.
    - **`$1`**: Refers to the first argument passed to the function, expected to be a URL.
    - **`$1 =~ ^https:\/\/huggingface\.co\/.*\.(safetensors|bin|ckpt|onnx|pt|pkl|yaml|yml|zip)$`**: This regex checks if the URL:
        - Starts with `https://huggingface.co/`.
        - Ends with one of the specified file extensions: `safetensors`, `bin`, `ckpt`, `onnx`, `pt`, `pkl`, `yaml`, `yml`, or `zip`.
    - If both conditions are true, it sets `auth_token` to the value of `HF_TOKEN` and `url_type` to `hf` to indicate it's a Hugging Face URL.

2. **Civitai URL Handling (Type 1):**

    ```bash
    elif [[ -n $CIVITAI_TOKEN && $1 =~ ^https:\/\/civitai\.com\/api\/download\/models\/[0-9]{1,6}$ ]]; then
        auth_token="$CIVITAI_TOKEN"
        url_type=civit1
    ```

    - **`-n $CIVITAI_TOKEN`**: Checks if the `CIVITAI_TOKEN` environment variable (Civitai authentication token) is set and not empty.
    - **`$1 =~ ^https:\/\/civitai\.com\/api\/download\/models\/[0-9]{1,6}$`**: This regex checks if the URL:
        - Starts with `https://civitai.com/api/download/models/`.
        - Is followed by 1 to 6 digits (a model ID).
        - Ends with no query string or additional characters.
    - If these conditions are true, it sets `auth_token` to `CIVITAI_TOKEN` and `url_type` to `civit1` to indicate it's a basic Civitai URL.

3. **Civitai URL Handling (Type 2):**

    ```bash
    elif [[ -n $CIVITAI_TOKEN && $1 =~ ^https:\/\/civitai\.com\/api\/download\/models\/[0-9]{1,6}\?(?:type=.*|&format=.*|&size=(full|pruned)|&fp=fp(16|32))+$ ]]; then
        auth_token="$CIVITAI_TOKEN"
        url_type=civit2
    ```

    - **`$1 =~ ^https:\/\/civitai\.com\/api\/download\/models\/[0-9]{1,6}\?(?:type=.*|&format=.*|&size=(full|pruned)|&fp=fp(16|32))+$`**: This regex checks if the URL:
        - Starts with `https://civitai.com/api/download/models/`.
        - Is followed by 1 to 6 digits (a model ID).
        - Includes query parameters for type, format, size (either `full` or `pruned`), and floating-point precision (`fp16` or `fp32`).
    - If these conditions are true, it sets `auth_token` to `CIVITAI_TOKEN` and `url_type` to `civit2` to indicate it's a more complex Civitai URL with additional query parameters.

4. **Download Execution:**

    ```bash
    if [[ ( -n $auth_token ) || ( $url_type=hf ) ]]; then
        wget --header="Authorization: Bearer $auth_token" -nc --content-disposition --show-progress -e dotbytes=4M -P "$2" "$1"
    elif [[ ( -n $auth_token ) || ( $url_type=civit1 ) ]]; then
        wget -nc --content-disposition --show-progress -e dotbytes=4M -P "$2" "$1?token=$auth_token"
    elif [[ ( -n $auth_token ) || ( $url_type=civit2 ) ]]; then
        wget -nc --content-disposition --show-progress -e dotbytes=4M -P "$2" "$1&token=$auth_token"
    else
        wget -nc --content-disposition --show-progress -e dotbytes=4M -P "$2" "$1"
    fi
    ```

    - **Hugging Face Downloads:**
        - If `auth_token` is set or `url_type` is `hf`, it uses `wget` with an `Authorization` header containing the Bearer token (`$auth_token`), which is required for authenticated downloads from Hugging Face.
        
    - **Civitai Downloads (Type 1):**
        - If `auth_token` is set or `url_type` is `civit1`, it appends `?token=$auth_token` to the URL for authenticated download, suitable for basic Civitai URLs.

    - **Civitai Downloads (Type 2):**
        - If `auth_token` is set or `url_type` is `civit2`, it appends `&token=$auth_token` to the URL to maintain the integrity of other query parameters for complex Civitai URLs.

    - **Unauthenticated Downloads:**
        - If none of the above conditions are met, it defaults to using `wget` without authentication, suitable for URLs that don't require tokens.

5. **`wget` Options Used:**
   - **`-nc`**: No clobber; prevents overwriting existing files.
   - **`--content-disposition`**: Uses the server-provided filename from the `Content-Disposition` header, if available.
   - **`--show-progress`**: Displays download progress in the terminal.
   - **`-e dotbytes=4M`**: Sets the progress dot size to 4 MB.
   - **`-P "$2"`**: Specifies the directory to save the downloaded file, where `$2` is the second argument passed to the function.

### Summary:

The `provisioning_download` function handles downloading files securely from Hugging Face and Civitai, using tokens for authentication where required. It distinguishes between basic and complex Civitai URLs based on the presence of specific query parameters and applies the appropriate authentication method. For each case, it uses `wget` with suitable options to manage authentication, handle progress reporting, and ensure safe file storage.
