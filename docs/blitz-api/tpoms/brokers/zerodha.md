# **Zerodha Adapter Documentation**

This guide explains how to connect your **Zerodha account** with **Blitz** using **Kite Connect APIs**, enabling secure authentication, market data access, and order execution directly through the platform.

---

## **Prerequisites**

Before you begin, ensure that:

- You have an active Zerodha trading account
- Kite Connect API access is enabled for your account

---

## **Create a Zerodha Developer Account**

1. Visit the Zerodha developer portal: [Zerodha Developer Portal](https://developers.kite.trade/signup)

   ![Zerodha Login Page](../../assets/zerodha/kiteloginpage.png)

2. Sign up or log in to create the account.
3. You will be redirected to the App creation page of Zerodha.

   ![Zerodha App Creation](../../assets/zerodha/app_creation.png)

4. Click on the Create new App button and choose Personal and fill the mandatory details

   ![App Credentials](../../assets/zerodha/app_credentials.png)

---

## **Get API Credentials**

After creating the app, Zerodha will provide:

- **API Key**
- **API Secret**

![App](../../assets/zerodha/test_app.png)

!!! warning "**Security Warning**"
    Keep these credentials secure. Do not expose them in public.

---

## **Generate TOTP Secret**

Login to the zerodha dashboard using the link [Zerodha Login](https://kite.zerodha.com/)

- Go to Password & Security at the top right corner of your dashboard

  ![Password and Security](../../assets/zerodha/password%20and%20security.png)

- Enable 2FA TOTP

  ![External 2FA TOTP enable](../../assets/zerodha/external%202FA%20TOTP%201.png)

- Download google authenticator app on your mobile and before scanning the QR code click on "Can't Scan? Copy the Code" at the bottom of the QR Code, and save the text this is your TOTP Secret

  ![Get TOTP Secret](../../assets/zerodha/external%202FA%20TOTP.png)

---

## **Fill all the credentials in the TPOMS**

After successfully generating all the credentials like

- API Key
- API Secret
- TOTP Secret

Login in to TPOMS

![TPOMS LOGIN](../../assets/zerodha/TPOMS.png)

Click the Connect button and you are connected

---
