# Elix
Elix is a voice-activated personal assistant in Python that opens websites, plays YouTube music, and responds with speech. Powered by speech recognition, TTS, and Selenium — your elixir of knowledge and control.
import speech_recognition as sr
import webbrowser
import pyttsx3
import wikipedia
import pyjokes
import datetime
import threading
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
import tkinter as tk

# === Core Setup ===
recogniser = sr.Recognizer()
engine = pyttsx3.init()

def speak(text):
    output_text.set(f"Elix: {text}")
    engine.say(text)
    engine.runAndWait()

# === GUI ===
window = tk.Tk()
window.title("Elix - Personal Voice Assistant")
window.geometry("400x200")
window.configure(bg="#121212")

status_text = tk.StringVar()
output_text = tk.StringVar()
status_text.set("Status: Waiting for wake word...")
output_text.set("Elix is ready to assist you!")

status_label = tk.Label(window, textvariable=status_text, font=("Arial", 12), fg="white", bg="#121212")
status_label.pack(pady=10)
output_label = tk.Label(window, textvariable=output_text, font=("Arial", 11), fg="#00ffcc", bg="#121212", wraplength=380, justify="center")
output_label.pack(pady=5)

# === Modular Command Handler ===
def processCommand(c):
    c = c.lower()

    if "time" in c:
        now = datetime.datetime.now().strftime("%I:%M %p")
        speak(f"The current time is {now}")
    elif "joke" in c:
        joke = pyjokes.get_joke()
        speak(joke)
    elif "wikipedia" in c:
        topic = c.replace("wikipedia", "").strip()
        try:
            summary = wikipedia.summary(topic, sentences=2)
            speak(summary)
        except:
            speak("Sorry, I couldn't find that on Wikipedia.")
    elif "play" in c:
        song = c.replace("play", "").strip()
        speak(f"Playing {song} on YouTube")
        playOnYouTube(song)
    elif "open" in c:
        openWebsite(c)
    elif "exit" in c or "bye" in c:
        speak("Goodbye!")
        window.quit()
    else:
        speak("I'm not sure how to respond to that.")

def openWebsite(c):
    sites = {
        "google": "https://www.google.com",
        "youtube": "https://www.youtube.com",
        "instagram": "https://www.instagram.com",
        "facebook": "https://www.facebook.com",
        "amazon": "https://www.amazon.in",
        "flipkart": "https://www.flipkart.com",
        "twitter": "https://www.twitter.com",
        "whatsapp": "https://web.whatsapp.com",
        "wikipedia": "https://www.wikipedia.org",
        "netflix": "https://www.netflix.com",
        "spotify": "https://open.spotify.com",
        "linkedin": "https://www.linkedin.com",
        "reddit": "https://www.reddit.com",
        "gmail": "https://mail.google.com",
        "chatgpt": "https://chat.openai.com"
    }

    for key in sites:
        if key in c:
            speak(f"Opening {key.capitalize()}")
            webbrowser.open(sites[key])
            return

    speak("I don't recognize that website.")

def playOnYouTube(song):
    query = "+".join(song.split())
    search_url = f"https://www.youtube.com/results?search_query={query}"

    options = Options()
    options.add_argument("--disable-infobars")
    options.add_argument("--start-maximized")
    options.add_argument("--disable-extensions")

    driver = webdriver.Chrome(options=options)
    driver.get(search_url)

    try:
        import time
        time.sleep(2)
        first_video = driver.find_element(By.ID, "video-title")
        first_video.click()
    except Exception as e:
        speak("Sorry, I couldn't play the video.")
        print("Error:", e)

# === Voice Listening Thread ===
def listen_loop():
    while True:
        try:
            with sr.Microphone() as source:
                status_text.set("Status: Listening for wake word...")
                audio = recogniser.listen(source, timeout=5, phrase_time_limit=3)
                word = recogniser.recognize_google(audio).lower()

                if "elix" in word:
                    speak("Yes?")
                    status_text.set("Status: Awaiting command...")
                    with sr.Microphone() as source:
                        audio = recogniser.listen(source, timeout=5, phrase_time_limit=6)
                        try:
                            command = recogniser.recognize_google(audio)
                            print("Command:", command)
                            processCommand(command)
                        except sr.UnknownValueError:
                            speak("Sorry, I didn't catch that.")
        except sr.UnknownValueError:
            continue
        except sr.WaitTimeoutError:
            continue
        except Exception as e:
            print("Listening Error:", e)

# Start background thread
listener_thread = threading.Thread(target=listen_loop)
listener_thread.daemon = True
listener_thread.start()

# Run the GUI
window.mainloop()
