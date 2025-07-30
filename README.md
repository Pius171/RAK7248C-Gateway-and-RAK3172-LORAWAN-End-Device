# RAK7248C-Gateway-and-RAK3172-LORAWAN-End-Device

### Step 1: Flashing Image to SD Card
I used Pi Imager to burn the image to an SD Card.
You can download the  Image file I burned to the SD card [here](https://downloads.rakwireless.com/LoRa/RAK7248C/Firmware/RAK7248C_Latest_Firmware.zip) 

### LORAWAN GATEWAY CONFIGURATIONS
For the initial setup of the gateway, refer to this documentation by RAK [here](https://docs.rakwireless.com/product-categories/wisgate/rak7248/quickstart/)

By default, the gateway is in AP mode. You would have noticed this if you followed the quick start guide above; however, I want the gateway to have internet access, so I put it in Client mode. This means it is connected to a router. I used my phone hotspot here, so I easily get its new IP address assigned to it by my phone from my phone's hotspot settings.

**Quick Note: When a device connects to a router, it is assigned an IP address, which is different from its IP address when it is in Access Point Mode.**

I then type in the new IP address it gets from my phone to PuTTY and  sign in via SSH using the default parameters.

---

### GATEWAY CONFIGURATION
Now in Client mode, type in the IP address into your browser like this <Gateway IP address>:8080 (Example: http://192.168.0.100:8080 ). Make sure your PC is also connected to the same router as the gateway, in this case, my phone.

You will be greeted with a login page:
<img width="1920" height="1046" alt="image" src="https://github.com/user-attachments/assets/7cb17c9a-c7f1-4b96-b451-169c63a2372d" />
The username and password is **admin**


#### Create a  Network Server
Click on Network Server on the right side of the screen and create a network server. 
* The name can be anything
* The server must be the same as seen in the image below
  <img width="975" height="223" alt="image" src="https://github.com/user-attachments/assets/1d34decd-56bf-42d1-9896-8fd8b0195381" />


---

#### CREATE A GATEWAY PROFILE
<img width="975" height="371" alt="image" src="https://github.com/user-attachments/assets/2720f3ff-9900-4339-addd-19f605892e8d" />

You can copy the same input I used in the image above, or you can put in what fits your scenario. Below is an image explaining a few things:
<img width="975" height="489" alt="image" src="https://github.com/user-attachments/assets/166179a6-dfbc-463a-956e-373f58c43740" />


---

#### SETTING UP SERVICE PROFILE
<img width="975" height="516" alt="image" src="https://github.com/user-attachments/assets/33fde550-677c-48ba-8356-a0f45fa83313" />

You should have something like this:
<img width="975" height="219" alt="image" src="https://github.com/user-attachments/assets/0516d468-1e6d-453b-9420-9cb2c5b5dc66" />

**Below are notes from ChatGPT to explain the parameters used in setting up a Service profile**

##### Notes from GPT

**What is Device Status Request Frequency?**

This is the number of times per day that ChirpStack will send a MAC command to your device requesting status info like battery level, margin, etc.

---

**What Should You Choose?**

Here are your options depending on your situation:

**✅ Best Option for RAK3172 + ABP (Manual/Testing Use)**  
Set it to `0`  
→ This disables automatic status requests.  
✅ Saves airtime  
✅ Prevents unnecessary downlink packets  
✅ Best for testing or if you don’t handle DevStatusReq in firmware

**🛠 If You Want Periodic Status (e.g., in Production)**  
Set it to `1` or `2` if:  
- You want once/twice a day health checks from devices  
- Your firmware handles the DevStatusReq MAC command  
- You’re monitoring a fleet of deployed nodes  

> But this only works well with OTAA, because ABP devices don’t always respond well unless specifically programmed to do so.

**💡 Recommendation for You**  
Since you're:  
- Using ABP  
- Testing/just trying to get messages through  
- Likely not parsing MAC commands on the RAK3172 yet  
→ Set it to `0`

---

I later switched to OTAA, meaning I set up the LoRaWAN End Device (RAK3172) to work using OTAA. This did not affect anything. If you read the Device Status Frequency above, you will realise why it won’t affect communication with the end device.

---

##### ADR (Adaptive Data Rate)

**🔧 What Do These Settings Mean?**

1. **Min Allowed Data Rate**  
   The lowest (slowest, longest range) data rate the Network Server is allowed to assign to the device.  
   - Lower DR = lower bandwidth but better range.

2. **Max Allowed Data Rate**  
   The highest (fastest, shorter range) data rate allowed.  
   - Data rates vary by region. For example, in EU868:  
     - DR0 = SF12 (long range, slow)  
     - DR5 = SF7 (shorter range, faster)

In ChirpStack:  
These settings limit the range of rates used by ADR (which adjusts the rate based on link quality).

---

**✅ What to Set for Testing**  
If:  
- You're using ABP (no join procedure)  
- Your RAK3172 sketch has a fixed DR  
- You don’t need ADR right now  

Then:  
- Leave both values as `0`, or don’t set them at all.  
→ This effectively means "don’t limit it" — ChirpStack will just go with what the device uses.

---

**🧠 Later on (Optional)**  
Once your messages are going through and you want:
- Efficient communication  
- Better battery life  
- Dynamic data rate switching  

You can:
- Enable ADR in your node’s firmware  
- Set min = 0 (SF12), max = 5 (SF7), or a range suited to your environment

---

⚠️ **Important**  
If your device doesn't handle ADR, ChirpStack may try to send MAC commands it can’t interpret.  
So, unless your firmware supports ADR:  
✅ Disable ADR on the device side and in ChirpStack. Leave those DR fields at `0`.

---

#### Creating a Device Profile
<img width="1289" height="664" alt="image" src="https://github.com/user-attachments/assets/6c0062c4-6cfb-4f35-b058-851cda1d0afe" />


##### Notes

- **Device Profile name**: any name you want  
- **Network server**: Already created when creating the network server  
- **LoRaWAN MAC Version**:
  - 📌 Why **1.0.3**?  
    RAK3172 uses the STM32WLE5 chip and, by default, supports LoRaWAN 1.0.3.  
    This version is commonly used in LoRaWAN deployments and is well supported in ChirpStack.  
    If you're using Arduino-based firmware, it usually targets 1.0.2 or 1.0.3.

- **LoRaWAN Regional Parameters Revision**:  
<img width="975" height="444" alt="image" src="https://github.com/user-attachments/assets/20fe167f-3226-4bd3-bcd1-15f758584bcb" />

- **ADR Algorithm**: Just choose the default  
- **MAX EIRP**:
  <img width="975" height="377" alt="image" src="https://github.com/user-attachments/assets/4b7153c2-2281-46b3-b784-45c4ed309921" />

- **Uplink Interval**:  
<img width="975" height="572" alt="image" src="https://github.com/user-attachments/assets/e1b9e73a-56e9-454e-92c7-c6b98f115a82" />
You can adjust it later 
---
<img width="975" height="494" alt="image" src="https://github.com/user-attachments/assets/4f91e761-2408-4144-9831-e11d993c5e45" />

### RX Configuration (Based on Lagos, Nigeria - EU868 Band)

| Field                     | Value            |
|--------------------------|------------------|
| RX1 Delay                | 5                |
| RX1 Data-Rate Offset     | 0                |
| RX2 Data-Rate            | 0                |
| RX2 Channel Frequency    | 869525000        |
| Factory Preset Frequencies | 868100000,868300000,868500000 |



---

**Note**: If you check the box **Device Supports OTAA**, you won’t need the above setup.

In the image below, you can ignore the tabs  in the red-bounded area:
<img width="975" height="192" alt="image" src="https://github.com/user-attachments/assets/20f0ac1e-c605-4f4a-9e80-bfd10df18bdd" />


---

#### Create a Gateway 
On the bottom right corner of the screen, click Gateway and then click  Create.
<img width="975" height="501" alt="image" src="https://github.com/user-attachments/assets/3560c359-4c78-4b3c-b0a7-d1e6c791268e" />

* You can give it any name and description
* To get your gateway ID. Go to the Putty terminal and type in `sudo gateway-version`. You should get a response with your gateway's ID
* Select the Network, Service and Gateway Profiles we created earlier

<img width="975" height="145" alt="image" src="https://github.com/user-attachments/assets/c570dd54-b0d3-4cb1-aa76-fc23f9752862" />

* There is no need to enable gateway discovery if you have only one gateway
* To get your approximate altitude, type in your location in ChatGPT and ask it to get your altitude. e.g Location: Lagos, Ikeja





#### Creating Application

Before setting up the device, you have to create an **Application**. Click on Applications in the bottom left corner of the ChirpStack web page.
<img width="1318" height="518" alt="image" src="https://github.com/user-attachments/assets/7bc2a8c5-38bd-4129-b7f5-35512ab7f371" />

* You can give it any name and description
* Select the service profile we created earlier.

#### Creating a Device
Now, click on the application you created.
<img width="1320" height="432" alt="image" src="https://github.com/user-attachments/assets/97316741-eeef-4a35-ad02-bfd53a929446" />
Click **Create** at the top right corner of the screen
<img width="975" height="458" alt="image" src="https://github.com/user-attachments/assets/d8b4b314-816b-4753-9ab4-bbe42b38fb3e" />


**To add a device**:

- The **device name and description** can be anything you want.
- The **device profile** is what we created earlier.
- **Generate a Device EUI** by clicking the refresh button beside **MSB**, and leave it as MSB (this is the format our Arduino code uses).
- Now, **copy that Device EUI** and add it to your Arduino code.
<img width="975" height="375" alt="image" src="https://github.com/user-attachments/assets/30de8d2d-1ed6-426f-8308-1d1c2ff2fbb8" />

---

Once you have created it, a page will appear asking for the **Application Key**.
<img width="975" height="207" alt="image" src="https://github.com/user-attachments/assets/0b9bbecc-91a1-4561-a782-d1d383d7fa67" />

You can generate it using the same process as above and then include it in your Arduino code
<img width="975" height="488" alt="image" src="https://github.com/user-attachments/assets/e0cbe17e-42f8-46b2-9f76-0c28ea216df8" />


### SETTING UP LORAWAN END DEVICE [RAK3172] USING ARDUINO
First, we need to install the RAK3172 board in the Arduino IDE.
* In the Arduino IDE, under files, preferences, add this board URL: https://raw.githubusercontent.com/RAKWireless/RAKwireless-Arduino-BSP-Index/main/package_rakwireless_com_rui_index.json and click ok
* Wait for Arduino to download the board.txt file, and from the board manager install the board below:
<img width="318" height="759" alt="image" src="https://github.com/user-attachments/assets/909f6548-7cd3-403a-9146-48d98831b42e" />
* Select the RAK3172 as your board in the Arduino IDE:
<img width="1193" height="708" alt="image" src="https://github.com/user-attachments/assets/5c6430c8-7c2c-4e08-951c-893228e9252b" />
* Open the OTAA example:
<img width="952" height="866" alt="image" src="https://github.com/user-attachments/assets/ff6a711f-7516-4c6d-870e-5be6c39bc7f6" />
* Edit the parameters with what you set up in the Gateway Config:
  <img width="927" height="91" alt="image" src="https://github.com/user-attachments/assets/eb207b23-8385-4024-9551-10532a2b8610" />

  **The APPEUI doesn't affect anything in OTAA mode, so it could just be anything, or all bits could be set to zero.**

  #### CONNECTING THE RAK3172
  <img width="1174" height="1080" alt="image" src="https://github.com/user-attachments/assets/bae61737-356b-42b8-9e9e-5218f0a60751" />
  <img width="1681" height="1080" alt="image" src="https://github.com/user-attachments/assets/d087c2ef-c015-4566-bb9b-1861841c274e" />

  **Make sure your USB to Serial converter is set to 3.3V**
  

  

  






