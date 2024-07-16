This project is about how we can enhance the usage of NFC technology with the help of python.

import tkinter as tk
from tkinter import ttk
from ttkthemes import ThemedTk
import serial
from googletrans import Translator, LANGUAGES
import pyttsx3
translator = Translator()
engine = pyttsx3.init()
root = ThemedTk(theme="arc")  # Use a ttk themed Tkinter window
root.title("NFC Tag Reader")
icon_path = "C:\\Users\\DELL\\Downloads\\logoforGUI.ico"
root.iconbitmap(icon_path)
serial_port = "COM16"  # Replace with your actual port
baud_rate = 115200
ser = serial.Serial(serial_port, baud_rate, timeout=1)
bg_color = "#303030"  # Dark grey 5
button_color = "#D2B48C"  # Light brown 3
def play_original_script():
    tag_info = message_label["text"].split("\n")[1].split(":")[-1].strip()  # Extracting the original message content
    engine.say(tag_info)
    engine.runAndWait()
def play_translated_script():
    try:
        if engine.isBusy():
            engine.stop()
        tag_info = message_label["text"].split("\n")[-1].split(":")[-1].strip()
        engine.say(tag_info)
        engine.runAndWait()
    except RuntimeError as e:
        print(f"Error: {e}")
def update_dropdown(event):
    entered_text = language_var.get().lower()
    matching_items = [lang for lang in language_options if lang.lower().startswith(entered_text)]
    language_menu['values'] = matching_items
    language_menu.focus_set()
    language_menu.event_generate('<Down>')
def set_speed(speed):
    engine.setProperty('rate', speed)
def set_speed_and_play():
    speed = float(speed_var.get())
    set_speed(speed)
    play_original_script()
def exit_gui():
    root.destroy()
def change_speed(speed_ratio):
    speed = float(speed_ratio)
    set_speed(speed)
    play_original_script()
style = ttk.Style()
style.configure("TLabel", background=bg_color, font=("Arial", 14), foreground="white")
style.configure("TButton", font=("Arial", 12), foreground="black", background=button_color)
message_label = ttk.Label(root, text="", padding=(10, 10))
message_label.pack(fill="both", expand=True)
language_options = list(LANGUAGES.values())
language_var = tk.StringVar()
language_menu = ttk.Combobox(root, values=language_options, textvariable=language_var, style="TCombobox")
language_menu.set(language_options[0])
language_menu.pack(pady=10)
language_menu.bind("<KeyRelease>", update_dropdown)
play_original_button = ttk.Button(root, text="Play Original Script", command=play_original_script, style="TButton")
play_translated_button = ttk.Button(root, text="Play Translated Script", command=play_translated_script, style="TButton")
exit_button = ttk.Button(root, text="Exit", command=exit_gui, style="TButton")
speed_buttons = [
    ttk.Button(root, text="0.75x", command=lambda: change_speed(0.75), style="TButton"),
    
]
play_original_button.pack(side="left", padx=10)
play_translated_button.pack(side="left", padx=10)
exit_button.pack(side="right", padx=10)
for button in speed_buttons:
    button.pack(side="left", padx=10)
tag_info_dict = {
    "99.46.182.18": "To your right you have Admin block",
    "195.233.59.146": "Go straight for Krishna Auditorium",
    "4.133.125.58": "Turn left for Block nine"
}
def read_tag_data():
    while True:
        line = ser.readline().decode("utf-8").strip()
        if line.startswith("tagId is :"):
            tag_id = line.split(":")[1].strip()
            translate_and_display_text(tag_id)
            play_original_script()  # Play the original script every time a card is read
def translate_and_display_text(tag_id=None):
    if tag_id:
        tag_info = tag_info_dict.get(tag_id, "Tag information not found")  # Handle missing tag IDs
        message_label["text"] = f"Tag ID: {tag_id}\nOriginal Message: {tag_info}"
    else:
        tag_info = message_label["text"].split("\n")[1].strip()
    translated_text = translator.translate(tag_info, dest=language_menu.get()).text
    message_label["text"] += f"\nTranslated Message: {translated_text}"
import threading
thread = threading.Thread(target=read_tag_data)
thread.start()
root.mainloop()
