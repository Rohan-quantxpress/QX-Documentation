# **IP Whitelisting**

IP whitelisting helps keep your app secure by allowing only the IP addresses you choose to connect to the Blitz Trader API. Any connection from an IP address that is not on your list will be blocked.

---

## Why Use IP Whitelisting?

Adding your IP address to the whitelist means that **only you** (and any other IPs you add) can use your app's API. This is a simple and effective way to stop others from accessing your account, even if your API key is somehow compromised.

---

## How to Add Your IP Address

1. Log in to the [BlitzTrader API Portal](https://{baseurl}/app/login).
2. Go to the **My Apps** section.
3. Click on your app to open it.
4. In the **IP whitelist** field, enter your IP address or addresses.
5. Click **Save** (or **Update**).

!!! tip "**Adding Multiple IPs**"
    You can add more than one IP address. Separate each address with a comma (`,`), a semicolon (`;`), a space, or simply put each one on a new line.

    **Example:**
    ```
    192.168.1.10, 192.168.1.11; 10.0.0.5
    ```

---

## How Do I Find My IP Address?

If you are not sure what your IP address is, search for **"What is my IP"** on any web browser — your public IP address will be shown at the top of the search results. Use this address in your whitelist.

!!! warning "**Check Your Connection**"
    Make sure you are connected through the same network you will be using when you run your app. If you connect from a different network later, you will need to add that network's IP address too.

---

## What Happens When My IP Is Not Whitelisted?

If a connection is made from an IP address that is not on your list, that request will be **blocked** and will not go through. This keeps your account safe from unauthorized access.

If you are unable to connect, the first thing to check is whether your current IP address is present in your app's IP whitelist.

---

## Managing Your Whitelist

- **Add an IP** — Edit your app and enter the new IP address in the **IP whitelist** field.
- **Remove an IP** — Delete the IP address you no longer need from the field and save your changes.
- **Edit other details** — You can manage your IP whitelist anytime from the **My Apps** page, just like any other app setting.

---

## Good Practices

- **Use a fixed IP** — If your internet connection uses a fixed (static) IP, add that address so your access does not keep breaking.
- **Keep it updated** — If your IP changes, update your whitelist right away so you do not lose access.
- **Add only what you need** — Add only the IP addresses you actually use. The fewer IPs on the list, the safer your app is.
