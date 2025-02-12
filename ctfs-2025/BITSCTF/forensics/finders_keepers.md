## Finders Keepers

We are given an image file called **weird.png**.

Using binwalk we see that there is hidden information in the image with the command:
`binwalk weird.png`

We extract it with the following command:
`binwalk --dd='.*' weird.png` 

We get the image **12005**

Then we use the foremost tool extract information if available of the 12005 image
`foremost 12005` 
 
We get a audio file **00000658.wav** that its clearly morse code and we get the string "snooooooppppppp"

Then we use it to extract more information in the **12005** image
`steghide extract -p "snooooooppppppp" -sf 12005` 

And we get the flag:
`BITSCTF{1_4m_5l33py_1256AE76}`
