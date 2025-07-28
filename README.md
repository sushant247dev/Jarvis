# Elix
Elix is a voice-activated personal assistant in Python that opens websites, plays YouTube music, and responds with speech. Powered by speech recognition, TTS, and Selenium — your elixir of knowledge and control.
import speech_recognition as sr
import webbrowser
import pyttsx3
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
import time

# Initialize recognizer and TTS engine
recogniser = sr.Recognizer()
engine = pyttsx3.init()

# Speak function
def speak(text):
    engine.say(text)
    engine.runAndWait()

# Handles commands
def processCommand(c):
    c = c.lower()

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
        if f"open {key}" in c or (key == "twitter" and "open x" in c):
            speak(f"Opening {key.capitalize()}")
            webbrowser.open(sites[key])
            return

    if "play" in c:
        song = c.replace("play", "").strip()
        speak(f"Playing {song} on YouTube")
        playOnYouTube(song)
        return

    if "exit" in c or "bye" in c:
        speak("Goodbye!")
        exit()

    speak("Sorry, I don't understand that command.")

# Play song using Selenium
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
        time.sleep(2)
        first_video = driver.find_element(By.ID, "video-title")
        first_video.click()
    except Exception as e:
        speak("Sorry, I couldn't play the video.")
        print("Error:", e)

# Main Program
if __name__ == "__main__":
    speak("Initializing Elix...")

    while True:
        try:
            with sr.Microphone() as source:
                print("Listening for wake word...")
                audio = recogniser.listen(source, timeout=3, phrase_time_limit=3)
                word = recogniser.recognize_google(audio).lower()

                if word == "elix":
                    speak("Yes?")
                    print("Elix activated...")

                    with sr.Microphone() as source:
                        audio = recogniser.listen(source, timeout=4, phrase_time_limit=5)
                        try:
                            command = recogniser.recognize_google(audio)
                            print("Command:", command)
                            processCommand(command)
                        except sr.UnknownValueError:
                            speak("Sorry, I didn't catch that.")
        except sr.UnknownValueError:
            pass
        except sr.WaitTimeoutError:
            pass
        except Exception as e:
            print("Error:", e)
