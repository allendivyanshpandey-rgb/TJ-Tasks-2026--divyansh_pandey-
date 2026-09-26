# TJ-Tasks-2026--divyansh_pandey-
AI Voice assistant using different python libraries such speech_recogination , gtts GTTS ,ollama, request(API) and os module.
listen - converts audio into text using speech_recogination library
by using ollama will generate a text reply for the audio which was initially converted into the text using speech_recogination library
and after that will use GTTS from gtts , playsound and os module to convert ollama text reply into read loud mp3 file .
will define different suitable functions for each above step and call the functions in the main() and iterate them using loop while(True) (condition).





[dry run code.zip](https://github.com/user-attachments/files/32667724/dry.run.code.zip)

import sounddevice as sd
import numpy as np
import speech_recognition as sr
import ollama
import requests
from gtts import gTTS
import playsound
import os


def suno(duration=5, fs=44100):
     #Record audio and convert speech to text.
    print(" Recording... (speak now)")
    recording = sd.rec(int(duration * fs), samplerate=fs, channels=1, dtype=np.int16)
    sd.wait()

  recognizer = sr.Recognizer()
    audio_data = sr.AudioData(recording.tobytes(), fs, 2)

  try:
        text = recognizer.recognize_google(audio_data)
        print(f"What You have just said: {text}")
        return text
    except sr.UnknownValueError:
        print(" Could not understand audio")
        return None
    except sr.RequestError:
        print(" Speech Recognition service error")
        return None


def puccho(comd):
     #Query local Ollama model.
    try:
        response = ollama.chat(
            model="llama3",
            messages=[{"role": "user", "content": comd}]
        )
        reply = response['message']['content']
        print(f"Ollama: {reply}")
        return reply
    except Exception as e:
        print(f" Ollama error: {e}")
        return "Local AI error."


def ask_openai(comd):
      #Query OpenAI cloud model securely.
    url = "https://api.openai.com/v1/chat/completions"
    api_key = os.getenv("OPENAI_API_KEY")  
    if not api_key:
        return " Missing API key. Set OPENAI_API_KEY in your environment."

   headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    data = {
        "model": "gpt-4o-mini",
        "messages": [{"role": "user", "content": comd}]
    }

  try:
        response = requests.post(url, headers=headers, json=data)
        if response.status_code == 200:
            result = response.json()
            reply = result["choices"][0]["message"]["content"]
            print(f"Cloud AI: {reply}")
            return reply
        else:
            return f" Error {response.status_code}: {response.text}"
    except Exception as e:
        return f"Cloud request failed: {e}"


def bolo(text):
     #Convert text to speech and play it
    try:
        tts = gTTS(text=text, lang="en")
        filename = "response.mp3"
        tts.save(filename)
        playsound.playsound(filename)
        os.remove(filename)
    except Exception as e:
        print(f"Speech error: {e}")


def main():
     #Main loop for the voice assistant.
    print(" Voice Assistant Ready! Say 'quit' to exit.")
    while True:
        user = listen()
        if not user:
            continue
        if user.lower() in ["quit", "exit", "stop"]:
            print(" Goodbye!")
            break

        
  if "cloud" in user.lower():
            ai_ans = ask_openai(user)
        else:
            ai_ans = puccho(user)

  bolo(ai_ans)


if __name__ == "__main__":
    main()




    
<img width="433" height="683" alt="screenshot 2026-09-26 at 11 54 06" src="https://github.com/user-attachments/assets/f01a1901-a4ef-4467-9b31-84dbdf91dd68" />
