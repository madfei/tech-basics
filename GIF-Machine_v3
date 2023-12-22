import tkinter as tk
from PIL import Image, ImageTk
import requests
from io import BytesIO
from tkinter import scrolledtext

# GIPHY API configuration
GIPHY_API_KEY = "2D2P3xA2sMtukT8PPqzKiUNqVzoPa92h"
GIPHY_API_ENDPOINT = "https://api.giphy.com/v1/gifs/random"

class MessageWidget(tk.Frame):
    def __init__(self, master=None, user_message="", gif_url=""):
        super().__init__(master)
        self.pack(pady=5, padx=10, side=tk.TOP, fill=tk.X)

        # Display the user's message
        self.user_label = tk.Label(self, text=user_message, bg="#DCF8C6", padx=10, pady=5, anchor="e", justify="right")
        self.user_label.pack(side=tk.TOP, fill=tk.X)

        # Display GIF
        self.gif_label = tk.Label(self, text="GIF will be displayed here", padx=10, pady=5, anchor="w", justify="left")
        self.gif_label.pack(side=tk.TOP, fill=tk.X)

        # Display GIF in the label
        self.display_gif(gif_url)

    def display_gif(self, gif_url):
        if gif_url:
            try:
                # Downloading the GIF
                response = requests.get(gif_url)
                response.raise_for_status()
                gif_data = BytesIO(response.content)

                # Create an animated PhotoImage object from the GIF data
                gif_image = Image.open(gif_data)
                gif_photo = ImageTk.PhotoImage(gif_image)

                # Update the GIF label with PhotoImage
                self.gif_label.config(text="", image=gif_photo, compound="left")
                self.gif_label.image = gif_photo  # Keep a reference to prevent garbage collection

                # Displayingnext frame after a delay (in milliseconds)
                self.after(100, lambda: self.display_next_frame(gif_image, 1, gif_photo))

            except requests.exceptions.RequestException as e:
                print(f"Error fetching GIF: {e}")
                self.gif_label.config(text="Error fetching GIF", image=None)

    def display_next_frame(self, gif_image, current_frame, gif_photo):
        try:
            # Get the current frame from the GIF
            gif_image.seek(current_frame)
            frame = gif_image.copy()

            # Create a new animated PhotoImage from the frame
            new_gif_photo = ImageTk.PhotoImage(frame)

            # Update the GIF label with the new PhotoImage
            self.gif_label.config(image=new_gif_photo, compound="left")
            self.gif_label.image = new_gif_photo  # Keep a reference to prevent garbage collection

            # Schedule the next frame after a delay (in milliseconds, 100 works well)
            self.after(100, lambda: self.display_next_frame(gif_image, current_frame + 1, new_gif_photo))

        except EOFError:
            # Restart the animation if we reach the end of the GIF
            self.after(100, lambda: self.display_next_frame(gif_image, 1, gif_photo))


class GifMessengerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("GIF Machine")

        # Create and configure GUI components
        self.create_gui()

        # Keep track of the current animation task
        self.current_animation_task = None

    def create_gui(self):
        # ScrolledText widget for displaying messages and GIFs
        self.message_text = scrolledtext.ScrolledText(self.root, wrap=tk.WORD, width=50, height=20)
        self.message_text.pack(pady=10, padx=10, side=tk.TOP, expand=True, fill=tk.BOTH)

        # Input field, send button
        self.input_field = tk.Entry(self.root)
        self.input_field.pack(pady=10, padx=10, side=tk.LEFT, expand=True, fill=tk.X)
        self.input_field.bind("<Return>", self.process_input)

        send_button = tk.Button(self.root, text="Send", command=self.process_input)
        send_button.pack(pady=10, padx=10, side=tk.RIGHT)

    def process_input(self, event=None):
        # Get user input
        user_input = self.input_field.get()

        # Get GIF URL
        gif_url = self.get_gif(user_input)

        # Display the sent message with GIF
        self.display_message(user_input, gif_url)

        # Clearing the input field after sending a message
        self.input_field.delete(0, tk.END)

    def get_gif(self, query):
        try:
            response = requests.get(GIPHY_API_ENDPOINT, params={"api_key": GIPHY_API_KEY, "tag": query})
            response.raise_for_status()
            data = response.json()

            if "data" in data and "images" in data["data"]:
                gif_url = data["data"]["images"]["original"]["url"]
                return gif_url
            else:
                print("Invalid response format from GIPHY API")
                return None

        except requests.exceptions.RequestException as e:
            print(f"Error fetching GIF: {e}")
            return None

    def display_message(self, user_message, gif_url):
        # Create a message widget and attach it to the ScrolledText widget
        message_widget = MessageWidget(self.root, user_message=user_message, gif_url=gif_url)
        self.message_text.window_create(tk.END, window=message_widget)

        # Scroll widget to the bottom to show the latest message
        self.message_text.yview(tk.END)


if __name__ == "__main__":
    root = tk.Tk()
    app = GifMessengerApp(root)
    root.mainloop()

