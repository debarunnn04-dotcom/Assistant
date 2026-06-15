






import speech_recognition as sr 
import webbrowser
import pyttsx3 

r = sr.Recognizer()
engine = pyttsx3.init()

voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)

engine.say("initializing google assistant...")
engine.runAndWait()



def processcommand(c):
   if "open google" in  c.lower():
       webbrowser.open("https://www.google.com")
   elif "open youtube" in c.lower():
       webbrowser.open("https://www.youtube.com")
   elif "open yes" in c.lower():
       webbrowser.open("https://www.chatgpt.com")
   elif "open i" in c.lower():
       webbrowser.open("https://www.instagram.com")

if __name__ == "__main__":
    pyttsx3.speak("initializing google assistant...")

    while True:
        try:
            with sr.Microphone() as source:
                print("Listening for wake word...")
                r.adjust_for_ambient_noise(source, duration=1)
                audio = r.listen(source)

            trigger = r.recognize_google(audio)
            print("wake word heard:", trigger)

            if any(word in trigger.lower() for word in ["google", "hey google", "ok google"]):
                print("Wake word matched")
                print("Speaking ya...")
                pyttsx3.speak("ya")

                with sr.Microphone() as source:
                    print("jarvis active...")
                    r.adjust_for_ambient_noise(source, duration=1)
                    audio = r.listen(source)

                command = r.recognize_google(audio)
                print("Command:", command)
                processcommand(command)
        except Exception as e:
            print("Error:", e)
