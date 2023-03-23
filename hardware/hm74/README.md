# Cyber Apocalypse 2023

## HM74

> As you venture further into the depths of the tomb, your communication with your team becomes increasingly disrupted by noise. Despite their attempts to encode the data packets, the errors persist and prove to be a formidable obstacle. Fortunately, you have the exact Verilog module used in both ends of the communication. Will you be able to discover a solution to overcome the communication disruptions and proceed with your mission?

>
>  Readme Author: [EnderNull](https://github.com/EnderNull)
>
> [`hw_hm74.zip`](hw_hm74.zip)

Provided: A docker instance is running that consistently outputs strings of binary information of 952 character length.  A .sv file is provided with the Verilog information pertaining to the construction of the communications device.  
Background: Verilog is a language used to program IC logic and describes the circuit to contain error correcting code known as HAM74 which is capable of correcting up to one bit of information per packet and recognizing two bits of error.
Solution: The Verilog programming seemed to use a non-standard formatting of the Ham 74 encoding, so the easiest method of accounting for the noise was to average the message across multiple transmissions with the assumption being that the noise would not regularly occur during the same sections of the transmissions.  The lines of the transmission would need to extract the data bits and form them into hex pairs for translation in CyberChef.

Script:
```
normalized_binary = ''
for i in range(len(binary_capture_1)):
	x = int(binary_capture_1[i]) + int(binary_capture_2[i]) + ...continues to 15... + int(binary_capture_15[i]) 

if x > 8:
	normalized_message += '1'
else:
	normalized_message += '0'

normalized_message_binary = ''
while normalized_binary != '':

# takes the last 2 packets of ham74 values
	ham74_section = normalized_binary[:14]
	normalized_message_binary += ham74_section[13]
	normalized_message_binary += ham74_section[12]
	normalized_message_binary += ham74_section[11]
	normalized_message_binary += ham74_section[9]
	normalized_message_binary += ham74_section[6]
	normalized_message_binary += ham74_section[5]
	normalized_message_binary += ham74_section[4]
	normalized_message_binary += ham74_section[2]

# removes the two packets from the string 
	normalized_binary = normalized_binary[14:]

print(normalized_message_binary)
```

## Flag
HTB{hmm_w1th_s0m3_ana1ys15_y0u_c4n_3x7ract_7h3_h4mmin9_7_4_3nc_fl49}