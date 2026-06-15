import webbrowser

import pyttsx3
import speech_recognition as sr


recognizer = sr.Recognizer()
microphone = sr.Microphone()

# Stop recording quickly after the user finishes speaking.
recognizer.pause_threshold = 0.25
recognizer.phrase_threshold = 0.05
recognizer.non_speaking_duration = 0.1
recognizer.dynamic_energy_threshold = True

GOOGLE_LANGUAGE = "en-IN"
PREFERRED_VOICE_NAMES = ("zira", "hazel", "heera", "female")


def speak(text):
    speaker = pyttsx3.init()
    voices = speaker.getProperty("voices")

    if voices:
        selected_voice = next(
            (
                voice
                for voice in voices
                if any(
                    name in voice.name.lower()
                    for name in PREFERRED_VOICE_NAMES
                )
            ),
            voices[1] if len(voices) > 1 else voices[0],
        )
        speaker.setProperty("voice", selected_voice.id)

    speaker.setProperty("volume", 1.0)
    speaker.setProperty("rate", 190)
    speaker.say(text)
    speaker.runAndWait()
    speaker.stop()


def listen(timeout=None, phrase_time_limit=4):
    with microphone as source:
        audio = recognizer.listen(
            source,
            timeout=timeout,
            phrase_time_limit=phrase_time_limit,
        )

    return recognizer.recognize_google(
        audio,
        language=GOOGLE_LANGUAGE,
    )


def process_command(command):
    normalized_command = command.lower()

    if "open google" in normalized_command:
        webbrowser.open("https://www.google.com")
        speak("Opening Google")
    elif "open youtube" in normalized_command:
        webbrowser.open("https://www.youtube.com")
        speak("Opening YouTube")
    elif "open chatgpt" in normalized_command:
        webbrowser.open("https://www.chatgpt.com")
        speak("Opening ChatGPT")
    elif "open instagram" in normalized_command:
        webbrowser.open("https://www.instagram.com")
        speak("Opening Instagram")
    else:
        speak("Sorry, I did not understand")


def run_assistant():
    wake_words = ("hey google", "ok google", "google")

    print("Quick microphone calibration...")
    with microphone as source:
        recognizer.adjust_for_ambient_noise(source, duration=0.15)

    speak("Initializing Google")

    while True:
        try:
            print("Listening...")
            speech = listen(phrase_time_limit=4)
            print("Heard:", speech)

            speech_lower = speech.lower()
            wake_word = next(
                (word for word in wake_words if word in speech_lower),
                None,
            )

            if not wake_word:
                continue

            # Fastest method: "Hey Google, open YouTube."
            command = speech_lower.split(wake_word, 1)[1].strip()

            if command:
                process_command(command)
                continue

            speak("Yeah")
            print("Listening for command...")
            command = listen(timeout=2, phrase_time_limit=4)
            print("Command:", command)
            process_command(command)

        except sr.WaitTimeoutError:
            print("No command heard.")
        except sr.UnknownValueError:
            print("Audio was not clear.")
        except sr.RequestError as error:
            print("Google speech service error:", error)
        except Exception as error:
            print("Error:", error)


if __name__ == "__main__":
    run_assistant()
