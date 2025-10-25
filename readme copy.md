# Cisco IP Phones on Asterisk/FreePBX using the [UseCallManager](https://usecallmanager.nz/) Patch

This repository contains a small collection of firmware, some exmple configuration files to make various models of Cisco IP phones work with an Asterisk-based PBX, particularly FreePBX.

## How It Works

The integration relies on a TFTP server to provision the Cisco phones and custom configurations on the Asterisk server to handle Cisco-specific SIP features.

1.  **Phone Boot-up**: When a Cisco phone boots, it requests its configuration file from a TFTP server using its MAC address.
2.  **TFTP Provisioning**: The TFTP server provides the phone with its configuration (`SEP<mac>.xml`), firmware, dialplan, ringtones, and background images.
3.  **SIP Registration**: The phone uses the information from its configuration file to register as a SIP extension on your Asterisk/FreePBX server.
4.  **Asterisk Configuration**: Custom Asterisk configuration files ensure that features like BLF (Busy Lamp Field), softkeys, and message waiting indicators (MWI) work correctly.

---

## Configuration Steps

### 1. TFTP Server Setup

You need to set up a TFTP server that is accessible to your phones. The root directory of your TFTP server should be populated with the contents of the following folders from this repository:

*   `Configuration Files/`
*   `Firmware/`
*   `Ringtones/`
*   `Desktops/`

**For each phone, you must:**

1.  Copy the `Configuration Files/SEPmac.cfn.xml` file.
2.  Rename the copy to `SEP<MAC_ADDRESS>.xml` (e.g., `SEPAABBCCDDEEFF.xml`).
3.  Edit the new `SEP<MAC_ADDRESS>.xml` file and replace the placeholder values for:
    *   Extension number
    *   SIP password (secret)
    *   Asterisk/FreePBX server IP address
    *   The correct firmware load file for the phone model (e.g., `SIP45.9-3-1SR4-1S.loads` for a 7945G).

### 2. Asterisk/FreePBX Custom Configuration

The files in `Asterisk & FreePBX/Asterisk Conf/` need to be integrated into your FreePBX installation. This is typically done by appending the contents of these files to the corresponding `_custom.conf` files in `/etc/asterisk/`.

*   **`sip_custom_post.conf`**: Contains settings for SIP peers (your phones) that are required for them to function correctly. This includes settings for codecs, transport, and context.
*   **`sip_notify_custom.conf`**: Defines a custom SIP NOTIFY for Cisco phones, which is essential for BLF and presence.
*   **`extensions_override_freepbx.conf`**: Used to override default FreePBX dialplan behavior to fix or enable specific features for the Cisco phones.
*   **`res_parking_custom.conf`**: Customizations for the parking lot application.

After adding these configurations, you need to reload Asterisk:
`rasterisk -x "core reload"`

### 3. The Asterisk Patch (Required for BLF)

**For full functionality, especially for Busy Lamp Field (BLF) and presence features, you must patch your Asterisk source code.**

*   **Why is it needed?**: Cisco's SIP implementation for presence (used for BLF) is not fully compliant with the standard that `chan_sip` in Asterisk expects. Without the patch, Asterisk cannot correctly process the SUBSCRIBE requests from the phones for presence information.
*   **How to get it?**: The patch is specific to the version of Asterisk you are running. You will need to search for a "Cisco BLF patch for chan_sip" that matches your Asterisk version. This usually involves downloading the Asterisk source, applying the patch, and recompiling `chan_sip.so`.

---

## Customization

*   **Ringtones**: Add `.raw` audio files to the `Ringtones/` directory and update `Ringtones/ringlist.xml` to make them available to the phones.
*   **Backgrounds**: Add background images (in `.png` format, correctly sized) to the appropriate sub-directory in `Desktops/` and update the corresponding `List.xml` file.
*   **Softkeys**: The `Configuration Files/SoftKeys.xml` file can be edited to create custom softkey layouts for different phone states (on hook, off hook, ringing, etc.).
