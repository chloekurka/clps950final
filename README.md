# clps950final
import gradio as gr
import openai
from dotenv import load_dotenv
load_dotenv()
import re


openai.api_key = "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"


def clean_dialogue(input_file, output_file):
    with open(input_file, 'r') as file:
        lines = file.readlines()

    cleaned_lines = []

    for line in lines:
        match = re.match(r'^"\d+"\s+"(.*?)"\s+"(.*?)"', line.strip())
        if match:
            character, dialogue = match.groups()
            cleaned_line = f"{character}: {dialogue}"
            cleaned_lines.append(cleaned_line)

    with open(output_file, 'w') as file:
        for line in cleaned_lines:
            file.write(line + '\n')

input_file = '/Users/chloe2/miniconda3/CLPS950/SW_EpisodeV.txt'
output_file = '/Users/chloe2/miniconda3/CLPS950/cleaned_SW.txt'

clean_dialogue(input_file, output_file)


def process_dialogue(file_paths, character1, character2):
    conversations = []

    for file_path in file_paths:
        with open(file_path, 'r') as file:
            lines = file.readlines()

        for line in lines:
            match = re.match(r'^(.*?):\s*(.*)$', line.strip())
            if match:
                name, content = match.groups()

                if name == character1:
                    role = 'user'
                elif name == character2:
                    role = 'system'
                else:
                    continue

                conversation = {
                    'role': role,
                    'content': content
                }
                conversations.append(conversation)

    return conversations


#file_paths = ['/Users/chloe2/miniconda3/CLPS950/S01E02.txt']
#character2 = 'Ross'
#character1 = 'Chandler'

#character2 = 'Phoebe'
#character1 = 'Joey'

file_paths = ['/Users/chloe2/miniconda3/CLPS950/cleaned_SW.txt']
#character2 = 'THREEPIO'
#character1 = 'VADER'

character2 = 'VADER'
character1 = 'THREEPIO'

messages = process_dialogue(file_paths, character1, character2)
#Friends characters
#messages.insert(0, {'role': 'system', 'content': "You are Phoebe Buffay from Friends. Pretend that you are her. Talk like Pheobe Buffay from the sitcom Friends. The following system messages are lines from Joey Tribbiani and the user messages are lines from Phoebe Buffay, to base your syntax off of. You should perform as though you are Phoebe and the user will be talking to Pheoebe as Joey."})
#messages.insert(0, {'role': 'system', 'content': "You are Ross Geller from Friends. Pretend that you are him. Talk like Ross Geller from the sitcom Friends. The following system messages are lines from Ross Geller and the user messages are lines from Chandler Bing, to base your syntax off of. You should perform as though you are Ross and the user will be talking to Ross as Chandler."})

#Star Wars characters
#messages.insert(0, {'role': 'system', 'content': "You are Threepio from Star Wars. Pretend that you are him. Talk like See-Threepio from Star Wars. The following system messages are lines from Threepio and the user messages are lines from Vader, to base your syntax off of"})
messages.insert(0, {'role': 'system', 'content': "You are Darth Vader from Star Wars. Pretend that you are him. Talk like Darth Vader from the Star Wars movies. The following system messages are lines from Darth Vader and the user messages are lines from See-Threepio, to base your syntax off of. You should perform as though you are See-Threepio and the user will be talking to See-Threepio as Darth Vader"})
print(messages)


def add_text(history, text):
    messages.append({"role": "user", "content": text })
    history = history + [(text, None)]
    return history, ""

def add_file(history, file):
    history = history + [((file.name,), None)]
    return history

def bot(history):
    user_message = history[-1][0]
    messages_with_user_input = messages.copy()
    messages_with_user_input.append({"role": "user", "content": user_message})

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=messages_with_user_input, 
        temperature=0.6
    )

    AImessage = response["choices"][0]["message"]["content"]
    history[-1][1] = AImessage
    return history


with gr.Blocks() as demo:
    chatbot = gr.Chatbot([], elem_id="chatbot").style(height=750)

    with gr.Row():
        with gr.Column(scale=0.85):
            txt = gr.Textbox(
                show_label=False,
                placeholder="Enter text and press enter, or upload an image",
            ).style(container=False)
        with gr.Column(scale=0.15, min_width=0):
            btn = gr.UploadButton("📁", file_types=["image", "video", "audio"])

    txt.submit(add_text, [chatbot, txt], [chatbot, txt]).then(
        bot, chatbot, chatbot
    )
    btn.upload(add_file, [chatbot, btn], [chatbot]).then(
        bot, chatbot, chatbot
    )



demo.launch()
