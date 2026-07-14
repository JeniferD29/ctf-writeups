## Introduction

**The Wool Files** is a simple, beginner-friendly digital forensics and steganography challenge. The room follows the investigation into the mysterious disappearance of Professor Woolsworth, with only three photographs left behind as evidence. Although the images appear ordinary at first, each contains hidden clues that can be uncovered using basic forensic and steganography techniques to reveal the final secret.

<img width="940" height="182" alt="image" src="https://github.com/user-attachments/assets/e968e124-f5c5-4454-bb2b-bb8e3479f115" />


## Task 1 — Office Investigation

After downloading the room files, extract the provided archive to access the evidence.


<img width="403" height="146" alt="image" src="https://github.com/user-attachments/assets/848ccbcd-5215-45b9-898e-126753aef0ab" />


Since the room focuses on digital forensics, the first step is to examine the metadata of the first image using `exiftool`.

```bash
exiftool IMG_4021.png
```

<img width="821" height="317" alt="image" src="https://github.com/user-attachments/assets/f2b7f08a-8817-4230-b0ba-73eb2f388ce2" />



The metadata reveals a comment containing Professor Woolsworth's favorite passphrase.


## Task 2 — The Barn Locker

Using the passphrase obtained from the metadata, we can extract the hidden data embedded inside `IMG_4022.jpg` using `steghide`.

```bash
steghide extract -sf IMG_4022.jpg
```

When prompted, enter the passphrase.

The extraction creates a file named `note.txt`. Display its contents using:

```bash
cat note.txt
```

The investigation log reveals that the final clue is hidden inside **IMG_4023.png**, directing us to continue the investigation with the third image.

> **Answer:** `IMG_4023.png`

<img width="940" height="438" alt="image" src="https://github.com/user-attachments/assets/7a4aa729-14be-4872-8afa-b294b1e99656" />



## Task 3 — The Final Evidence

The note indicates that the final evidence is hidden within `IMG_4023.png`. To inspect the image for embedded files, use `binwalk`.

```bash
binwalk -e IMG_4023.png
```

<img width="940" height="143" alt="image" src="https://github.com/user-attachments/assets/fe245465-d121-4969-b751-ce45919de214" />

The output reveals an embedded ZIP archive containing `flag.txt`. Navigate to the extracted directory and list its contents.

```bash
cd _IMG_4023.png.extracted
ls
```

Among the extracted files is `flag.txt`. Display its contents to obtain the final flag.

```bash
cat flag.txt
```

```text
flag
```

<img width="811" height="266" alt="image" src="https://github.com/user-attachments/assets/d8e70dac-70ba-4a8c-af03-b602c17dd627" />
