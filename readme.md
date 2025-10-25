# Cisco IP Phones on Asterisk/FreePBX using the [UseCallManager](https://usecallmanager.nz/) Patch

This repository contains a small collection of firmware, some exmple configuration files to make various models of Cisco IP phones work with an Asterisk-based PBX, particularly FreePBX.

> Self-Promo: [Setup the Cisco 7911 using chan_pjsip and a TFTP Server for this and the other phones. (Click to open my blog post!)](https://pintobernardo.wordpress.com/2025/06/15/setting-up-cisco-7911-for-freepbx-asterisk-sip/)

---

## Repository Contents

This repository is organized into the following directories:

| Directory              | Purpose                                                                              |
| :--------------------- | :----------------------------------------------------------------------------------- |
| `Asterisk & FreePBX`   | Holds Asterisk `.conf` files for SIP, presence, and custom feature integration.      |
| `Configuration Files`  | Contains XML templates for phone provisioning (e.g., `SEP<mac>.xml`, `dialplan.xml`). |
| `Desktops`             | Stores phone background images, sorted by screen resolution.                         |
| `Firmware`             | A collection of SIP and SCCP firmware for various Cisco phone models.                |
| `Ringtones`            | Contains `.raw` audio files for ringtones and the `ringlist.xml` configuration.      |

---

## TFTP Server Directory Example

For the phones to work, your TFTP server's root directory must contain all the necessary files. You should **not** create `/Firmware/` or `/Ringtones/` subdirectories in your TFTP root.

Instead, copy the *contents* of the `Firmware` and `Ringtones` folders from this repository directly into your TFTP root. The `Desktops` folder, however, should be copied as a complete folder.

**Example of a correct TFTP root directory structure:**

```
/tftpboot/
│
├── SEP001122334455.xml
├── SEPAABBCCDDEEFF.xml
├── dialplan.xml
├── SoftKeys.xml
│
├── Desktops/                <-- This folder and its contents are copied directly
│   ├── 320x196x4/
│   │   ├── image1.png
│   │   └── ...
│   └── List.xml
│
├── SIP45.9-3-1SR4-1S.loads  <-- Firmware files are in the root
├── apps45.9-3-1ES26.sbn
├── cnu45.9-3-1ES26.sbn
│
├── Analog1.raw              <-- Ringtone files are in the root
├── Classic2.raw
├── ringlist.xml

```

---

## ⚠️ Important Warning: Patch & Driver Requirements

To achieve full functionality with these phones (especially for features like BLF), you must use the **`UseCallManager` patch**.

This patch **requires the legacy `chan_sip` driver**.

Modern Asterisk versions (21+) have completely removed `chan_sip`, so you **must run Asterisk 20 or older**. This process is complex and requires more than just enabling a setting. You will need to download the Asterisk source code that matches your version, apply the `UseCallManager` patch to the `chan_sip` module, and then recompile and install it.

> **A Note of Thanks**
> The process of patching and recompiling a custom Asterisk module can be complex. We are grateful for community members who create guides on this topic. The following guide provides an excellent walkthrough.
> 
> **Recommended Guide:** [**FreePBX v17 on Debian v12 with Asterisk v20 & UseCallManager Cisco Patch**](https://christai.com/asterisk/709-01-freepbx-v17-debian-v12-asterisk-v20-usecallmanager-cisco-patch.html)
> 
> **Important Notes:**
> * It is recommended to visit the official Asterisk and UseCallManager GitHub pages to ensure you are downloading the correct, most up-to-date, and supported versions for your environment.
> * The author of this repository has only tested this process with Asterisk version `20.15.12`.

---

## Configuring the `SEP<MAC>.xml` File

For each phone you want to provision, you must create a specific configuration file for it.

1.  Make a copy of `Configuration Files/SEPmac.cfn.xml`.
2.  Rename the copy to `SEP<MAC_ADDRESS>.xml`, where `<MAC_ADDRESS>` is the 12-character MAC address of your phone (e.g., `SEPAABBCCDDEEFF.xml`).
3.  Open the new file and edit the placeholder values. All fields you need to change are marked with `{===}`.

### Complete List of Configuration Fields

| Path in XML                                 | Field                | Description                                                              |
| :------------------------------------------ | :------------------- | :----------------------------------------------------------------------- |
| `.../ntps/ntp/name`                         | NTP Server           | The address of your Network Time Protocol (NTP) server.                  |
| `.../callManager/processNodeName`           | Asterisk Server IP   | The IP address of your Asterisk/FreePBX server.                          |
| `.../sipProfile/phoneLabel`                 | Phone Screen Label   | Text shown in the top-right of the phone screen (e.g., extension).       |
| `.../sipLines/line/featureLabel`            | Line Button Label    | The label displayed next to the primary line button.                     |
| `.../sipLines/line/displayName`             | Display Name         | The Caller ID name that will be shown to other phones.                   |
| `.../sipLines/line/name`                    | Extension Number     | The SIP extension number used for registration and dialing.              |
| `.../sipLines/line/authName`                | Auth Username        | The authentication username for the SIP extension.                       |
| `.../sipLines/line/authPassword`            | Auth Password        | The secret/password for the SIP extension.                               |
| `.../sipLines/line/contact`                 | Contact Header       | The SIP contact for the line, typically the extension number.            |
| `.../loadInformation`                       | Firmware Load File   | The specific `.loads` file name for the phone's model.                   |
| `.../servicesURL`                           | Services URL         | URL for Cisco XML phone services (e.g., `http://<server_ip>/services.php`). |
| `.../directoryURL`                          | Directory URL        | URL for a corporate directory XML file (e.g., `http://<server_ip>/directory.xml`). |
| `.../informationURL`                        | Information URL      | URL for an information service (e.g., weather, news).                    |

---

## Further `SEPmac.xml` Customization

The `SEPmac.cnf.xml` file is highly customizable, allowing for multiple lines, speed dials, BLF (Busy Lamp Field) buttons, and many other advanced features.

For a comprehensive guide on all the available options and how to configure them, please refer to the official documentation from UseCallManager.

*   **Official Documentation:** [**SEPmac.cnf.xml File Explained**](https://usecallmanager.nz/sepmac-cnf-xml.html)

---

## Customization & Configuration Guide

This section provides details on how to customize various features of the phones.

### 1. Asterisk/FreePBX Configuration (`.conf` files)

These files should be placed in your Asterisk configuration directory (typically `/etc/asterisk/`). You may need to append the contents of these files to your existing `_custom.conf` files in FreePBX.

*   **`sip_custom_post.conf`**: Defines the SIP peer configurations for your Cisco phones. The `[cisco]` template enables Cisco-specific features from the patch. For each phone, you must add a block defining its extension and telling it to subscribe for presence information.
    ```ini
    [{==Extension==}](+,cisco)
    subscribe={==Extension==}
    ```
    For more details, see the [SIP Peers Documentation](https://usecallmanager.nz/sip-conf.html).

*   **`sip_notify_custom.conf`**: Defines custom SIP NOTIFY commands that can be sent to the phones for remote management, such as `cisco-restart` and `cisco-reset`. You generally do not need to edit this file.

    For more details, see the [SIP Notify Commands Documentation](https://usecallmanager.nz/sip-notify-conf.html).

*   **`extension_custom.conf`**: Contains the custom dialplan logic (extensions) that powers phone features like call-forwarding, pickup, parking, paging, intercom, and voicemail. You generally do not need to edit this file for basic functionality.

    For more details, see the [Dialplan Extensions Documentation](https://usecallmanager.nz/extensions-conf.html).

### 2. Phone Configuration Files

These XML files are placed in the TFTP root directory and control phone-side features.

*   **`dialplan.xml`**: This file controls the phone's own dialing behavior, such as how long to wait after digits are pressed. The version in this repository is customized for Portugal. You may need to edit the `<TEMPLATE>` patterns to match your local dialing rules.

    For more information, see the [Dial Template Documentation](https://usecallmanager.nz/dial-template-xml.html).

*   **`SoftKeys.xml`**: This file defines the softkey layouts for different call states (e.g., `On Hook`, `Connected`). You can customize this file by changing which keys are assigned to each `<softKeySet>`.

    For more information, see the [Soft Keys XML Documentation](https://usecallmanager.nz/soft-keys-xml.html).

### 3. Custom Ringtones

You can add your own ringtones to the phones by following these steps:

1.  **Convert your audio file.** The file must be in the **G.711 u-law, 8000 Hz, 8-bit, mono** format, with a `.raw` extension.
2.  Place your new `.raw` file in the TFTP root directory.
3.  Open `ringlist.xml` and add a new `<Ring>` entry:

    ```xml
    <Ring>
        <DisplayName>My Custom Ring</DisplayName>
        <FileName>my_custom_ring.raw</FileName>
    </Ring>
    ```
    For more details, see the [Ring List XML Documentation](https://usecallmanager.nz/ring-list-xml.html).

### 4. Custom Desktop Backgrounds

You can add custom background images for different phone models.

First, find your phone model in the table below to determine the correct image dimensions. The `Depth` refers to the color depth in bits.

| Model                        | Type      | Width | Height | Depth | Preview Width | Preview Height | Directory               |
| :--------------------------- | :-------- | :---- | :----- | :---- | :------------ | :------------- | :---------------------- |
| 7911, 7906                   | Greyscale | 95    | 34     | 1     | 23            | 8              | Desktops/95x34x1        |
| 7941, 7961, 7942, 7962       | Greyscale | 320   | 196    | 4     | 80            | 53             | Desktops/320x196x4      |
| 7945, 7965                   | Color     | 320   | 212    | 16    | 80            | 53             | Desktops/320x212x16     |
| 7970, 7971, CIPC             | Color     | 320   | 212    | 12    | 80            | 53             | Desktops/320x212x12     |
| 7975                         | Color     | 320   | 216    | 16    | 80            | 53             | Desktops/320x216x16     |
| 8821                         | Color     | 240   | 320    | 24    | 117           | 117            | Desktops/240x320x24     |
| 8841, 8845, 8851, 8861, 8865 | Color     | 800   | 480    | 24    | 139           | 109            | Desktops/800x480x24     |
| 8941, 8945                   | Color     | 640   | 480    | 24    | 123           | 111            | Desktops/800x480x24     |
| 8961, 9951, 9971             | Color     | 640   | 480    | 24    | 123           | 111            | Desktops/640x480x24     |

**To add a new image:**

1.  Create your image in `.png` format with the correct dimensions from the table.
2.  Create a smaller `.png` thumbnail image (using the `Preview Width` and `Preview Height` dimensions).
3.  Place both files in the correct directory inside the `Desktops` folder.
4.  Open the `List.xml` file within that same directory.
5.  Add a new `<ImageItem>` entry that points to your thumbnail and full-size image.

    ```xml
    <ImageItem Image="TFTP:Desktops/320x196x4/My-Thumb.png" URL="TFTP:Desktops/320x196x4/My-Image.png"/>
    ```

For more details, see the [Image List XML Documentation](https://usecallmanager.nz/image-list-xml.html).

---

## Firmware Disclaimer

The Cisco firmware files included in this repository are the intellectual property of Cisco Systems, Inc. They are provided here for convenience and for use with personal and lab equipment. No ownership is claimed. If you are a representative of Cisco Systems, Inc. and wish for this firmware to be removed, please open an issue on this repository.