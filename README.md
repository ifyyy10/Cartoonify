import cv2
import pathlib
import pyautogui
from tkinter import *
from PIL import ImageTk, Image
from tkinter import filedialog

class Cartoonified_Image:
    def __init__(self, root):
        self.window = root
        self.window.geometry("960x560")
        self.window.title("Ifeanyi's Cartoonizer")
        self.window.resizable(width = False, height = False)

        self.width = 820
        self.height = 520

        self.Image_Path = ''

    
        # ================Menubar Section===============
    
        # Creating Menubar
        self.menubar = Menu(self.window)

        # Menu to open file to select image to cartoonify
        open = Menu(self.menubar, tearoff=0)
        self.menubar.add_cascade(label='Open', menu=open)
        open.add_command(label='Open Image',command=self.open_image)
        
        
        # Menu widget to cartoonify the image
        cartoonify = Menu(self.menubar, tearoff=0)
        self.menubar.add_cascade(label='Cartoonify', menu=cartoonify)
        cartoonify.add_command(label='Create Cartoon', command=self.cartoonify)
        
        # Menu to exit the Application
        exit = Menu(self.menubar, tearoff=0)
        self.menubar.add_cascade(label='Exit', menu=exit)
        exit.add_command(label='Exit Cartoonizer', command=self._exit)

        # Configuring the menubar
        self.window.config(menu=self.menubar)

        # ===================End=======================


        # Creating a Frame which places the image to be cartoonized at the center. 
        # The image height and width passed as variables are also defined as well 
        self.frame = Frame(self.window, width=self.width, height=self.height)
        self.frame.pack()
        self.frame.place(anchor='center', relx=0.5, rely=0.5)

    # Open an Image through filedialog(allows you to specify a file to open or save)
    def open_image(self):
        self.clear_screen()
        self.Image_Path = filedialog.askopenfilename(title = "Select an Image", filetypes = (("Image files", "*.jpg *.jpeg *.png"),))
        if len(self.Image_Path) != 0:
            self.show_image(self.Image_Path)
    
    # Open and Display the Image
    def show_image(self, img):
        # Opening the image
        image = Image.open(img)
        # resize the image, so that it fits to the screen
        resized_image = image.resize((self.width, self.height))

        # Create an object of tkinter ImageTk becuase tkinter ALWAYS expects an image object
        # purpose is to return the image height and weight in pixels
        self.img = ImageTk.PhotoImage(resized_image)

        # A Label Widget for displaying the Image. This actually displays the frame and the image
        label = Label(self.frame, image=self.img)
        label.pack()

    def cartoonify(self):
        # Storing the image path(selected image) to a variable
        ImgPath = self.Image_Path

        # If any image is not selected 
        if len(ImgPath) == 0:
            pass
        else:
            # Get the file name to be saved after cartoonify the image
            filename = pyautogui.prompt("Enter the filename to be saved")
            # Filename with the extension (extension of the selected image)
            filename = filename + pathlib.Path(ImgPath).suffix
            # Load the image to cartoonize        
            img = cv2.imread(ImgPath)
            img = cv2.resize(img, (840,680))

            # Apply a bilateral filter to smoothen the image while preserving edges.
            smoothened_image = cv2.bilateralFilter(src=img, d=9, sigmaColor=75, sigmaSpace=75)

            #Convert image to gray and create an edge mask using adaptive thresholding.
            gray_img = cv2.cvtColor(src=smoothened_image, code=cv2.COLOR_BGR2GRAY)
            edges = cv2.adaptiveThreshold(src=gray_img, maxValue=255, adaptiveMethod=cv2.ADAPTIVE_THRESH_MEAN_C, thresholdType=cv2.THRESH_BINARY, blockSize=9, C=9)
            
            # Combine the smoothened image and the edge mask to create a cartoon-like effect.
            cartoonized_img = cv2.bitwise_and(src1=smoothened_image, src2=smoothened_image, mask=edges)
        
            # Save the cartoon image in our cartooned_image directory.
            # Note that you must have a folder named cartooned_image in the project directory for the image to be saved and displayed
            cv2.imwrite(f'cartooned_image/{filename}', cartoonized_img)

            #Clear screen and display the cartooned image.
            self.clear_screen()
            self.show_image(f'cartooned_image/{filename}')

    # Remove all widgets from the frame
    def clear_screen(self):
        for widget in self.frame.winfo_children():
            widget.destroy()

    # It destroys the main GUI window of the
    # application
    def _exit(self):
        self.window.destroy()

# The main function
# If the script or our python file is run as the main program, then the code will be executed(seperates executable codes from impoted module codes)
if __name__ == "__main__":
    # initialize the interpreter or tkinter module and create a root window
    root = Tk()
    # Creating an object of Image_Cartoonify class
    obj = Cartoonified_Image(root)
    
    root.mainloop()



        
