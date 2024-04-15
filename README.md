# GPT-4 Query Jupyter Notebook

This is a simple-yet-powerful alternative to ChatGPT. It was created to easily access the OpenAI API [Text Completions](https://platform.openai.com/docs/guides/text-generation) endpoints (aka the GPT models) without having to use curl.

### Install dependencies.

1. Create a python virtual environment: `python3 -m venv .venv`

2. Activate the virtual envirnment: `source .venv/bin/activate`

3. Install the required dependencies: `pip install -r requirements.txt`

4. Create a .env file to store your API secret: `echo 'OPENAI_API_KEY="##########"' >> .env`


```python
from dotenv import load_dotenv
from openai import OpenAI
from pathlib import Path
from os import getenv, mkdir, getcwd, system
```

### Declare configuration constants.


```python
SYSTEM_CONFIG = {
    'CWD': Path(getcwd()),
    'ENCODING': 'utf-8'
}

GPT_CONFIG = {
    'MODEL': 'gpt-4-turbo-preview',
    'SYSTEM_PROMPT_PATH': SYSTEM_CONFIG['CWD'] / 'input' / 'system_prompt.txt',
    'USER_PROMPT_PATH': SYSTEM_CONFIG['CWD'] / 'input' / 'user_prompt.txt',
    'OUTPUT_FOLDER_PATH': SYSTEM_CONFIG['CWD'] / 'output',
    'DEFAULT_SYS_PROMPT': """#PERSONA
You are an experienced problem solver.
#END

#INSTRUCTIONS
1. You are given instructions, delimited by XML tags.
2. Execute the insturctions and respond accordingly.
#END""",
    'DEFAULT_USR_PROMPT': """<instructions>
</instructions>""",
}

PROMPT_PATHS = (GPT_CONFIG['SYSTEM_PROMPT_PATH'], GPT_CONFIG['USER_PROMPT_PATH'])
```

### Load OpenAI API Key


```python
load_dotenv()
api_key = getenv('OPENAI_API_KEY')
```

### Load prompts from local textfiles.


```python
def load_prompts_from_textfiles(prompt_paths: tuple) -> tuple:
     with open(prompt_paths[0], 'r', encoding=SYSTEM_CONFIG['ENCODING']) as file:
          sys = file.read()
     with open(prompt_paths[1], 'r', encoding=SYSTEM_CONFIG['ENCODING']) as file:
          usr = file.read()
     prompts = (sys, usr)
     return prompts

try:
     prompts = load_prompts_from_textfiles(PROMPT_PATHS)
except FileNotFoundError as e: # ERROR HANDLING IF INPUT FOLDER DOES NOT EXIST.
     print(f"Caught: {e}")
     try:
          mkdir("input")
     except FileExistsError:
          print("Error: 'input' folder already exists. Delete to run this script.")
     # build paths
     input_path = SYSTEM_CONFIG["CWD"] / 'input'
     sys_input_path = input_path / 'system_prompt.txt'
     usr_input_path = input_path / 'user_prompt.txt'
     # Native calls to echo
     system(f"echo \"{GPT_CONFIG['DEFAULT_SYS_PROMPT']}\" >> {sys_input_path}")     
     system(f"echo \"{GPT_CONFIG['DEFAULT_USR_PROMPT']}\" >> {usr_input_path}")
     # Reminder to user to edit prompts
     print(f"You must edit prompt files: `{sys_input_path}` and `{usr_input_path}`")
```

### Build message for API.


```python
def message_builder(prompts: tuple) -> list:
    msg = [
        {"role": "system", "content": prompts[0]},
        {"role": "user", "content": prompts[1]}
    ]
    return msg
```

### Query API and store response. Must be ran twice.


```python
count = 0
response = None
response_text = ""
if (count == 0):
    print("Run again to confirm.")
elif (count == 1):
    # Run code here
    client = OpenAI()
    print("Querying GPT model. This may take a while...")
    response = client.chat.completions.create(model=GPT_CONFIG['MODEL'], messages=message_builder())
    response_text = response.choices[0].message.content
    count = 0
else:
    print("Error: Restart notebook.")
```

    Run again to confirm.


### Create response filename


```python
filename = response_text[:20]
filename = filename + ".md"
filename = filename.replace(" ", "_")

file_path = GPT_CONFIG['OUTPUT_FOLDER_PATH'] / filename

print(filename)
```

    .md


### Write response to file


```python
def write_file(file_path: Path, file_content: str) -> None:
    with open(file_path, 'w') as file:
        file.write(file_content)
    print(f"Response written to {file_path}.")


try:
    write_file(file_path, response_text)
except FileNotFoundError as e:
    print(f"Warn: {e}")
    mkdir(GPT_CONFIG['OUTPUT_FOLDER_PATH'])
    write_file(file_path, response_text)

```

    Response written to /Users/nickkammerer/Documents/dev/jupyter-gpt/output/.md.


(WIP) Using pandoc, write the markdown to pdf.


```python
#pandoc_output_file_path = str(file_path).split(".")[0] + '.pdf'
#system_call = f"pandoc {file_path} -o {pandoc_output_file_path}"
#
#try:
#    system("")
#except Exception as e:
#    print(e)
```
