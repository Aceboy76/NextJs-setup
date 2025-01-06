<h1 align="center"> Next Js Setup </h1> <br>
<p align="center">
    <img alt="NextJSLogo" title="NextJSLogo" src="https://logowik.com/content/uploads/images/nextjs7685.logowik.com.webp" width="450">
</p>

<p align="center">
  Make an easier setup for NextJs
</p>



<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Create Folder](#Create-Folder)
- [Folder Heirarchy](#Folder-Heirarchy)
- [Install Packages](#Install-packages)
- [Edit Files](#Edit-Files)
- [Optional](#Optional)
- [Backers](#backers-)
- [Sponsors](#sponsors-)
- [Acknowledgments](#acknowledgments)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Create Folder

<p>First open terminal and create a folder wherever you like.</p>
<p>(i use bun for installment, you can use npm if you like).</p>
<p>check the installment for nextjs with npm.</p>

```
cd .\Desktop\

mkdir NextJs_Project

bun create next-app
```          

## Folder Heirarchy
<p>The Folder should look like this in the vscode</p>

```
app/
├─ (public)/
│  ├─ HomePage/
│  │  ├─ page.tsx
│  │  ├─ layout.tsx
├─ api/
│  ├─ user/
│  │  ├─ action.ts
lib/
├─ session,ts
├─ lib,ts
components/
├─ ui/
│  ├─ button.tsx
├─ navbar.tsx
env

```


## Install Packages
<p>Install this necessary packeges</p>

```
bun install tailwind-merge
bun install bcrypt-ts
bun install zod
bun install jose
npx shadcn-ui@latest init
```

## Edit Files

<p>Change the  layout.tsx file to this</p>

```
import "./globals.css";

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body
        className={`antialiased`}
      >
        {children}

      </body>
    </html>
  );
}

```
<p>Add this to the lib.ts</p>

```
import { promises } from "dns";
import { SignJWT, jwtVerify } from "jose";

const secretKey = process.env.JWT_SECRET_KEY;
const key = new TextEncoder().encode(secretKey);

// eslint-disable-next-line @typescript-eslint/no-explicit-any
export async function encrypt(payload: any) {
  return await new SignJWT(payload)
    .setProtectedHeader({ alg: "HS256" })
    .setIssuedAt()
    .setExpirationTime("1h")
    .sign(key);
}
// eslint-disable-next-line @typescript-eslint/no-explicit-any
export async function decrypt(session: string): Promise<any> {
  try {
    const { payload } = await jwtVerify(session, key, {
      algorithms: ['HS256'],
    })
    return payload
  } catch (error) {
    console.log('Failed to verify session')
  }
}
```

<p>Add this to the session.ts</p>

```
import { cookies } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

import { decrypt, encrypt } from "./lib";


export async function getSession() {
    const session = (await cookies()).get("session")?.value;
    if (!session) return null;
    return await decrypt(session);
}


export async function updateSession(request: NextRequest) {
    const session = request.cookies.get("session")?.value;
    if (!session) return;

    const parsed = await decrypt(session);
    if (!parsed) return;

    parsed.expires = new Date(Date.now() + 1 * 60 * 60 * 1000);

    const res = NextResponse.next();
    res.cookies.set({
        name: "session",
        value: await encrypt(parsed),
        httpOnly: true,
        expires: new Date(parsed.expires),
    });
    return res;
}

export async function deleteSession() {
    (await cookies()).set("session", "", { expires: new Date(0) });
}

```


## Optional

<p>You can also create email verification with nodemail</p>

<p>first install nodemailer</p>

```
bun install nodemailer
```

<p>add this to you env file</p>

```
JWT_SECRET_KEY="generatedSecretkey"
EMAIL_HOST="smtp.gmail.com"
EMAIL_PORT="465"
EMAIL_USER="youremail"
EMAIL_PASS="yourpassword"

```

<p>Then create a file for sending email verification</p>

```
'use server'

import nodemailer from 'nodemailer';
import { decrypt, encrypt } from '@/lib/lib';
import prisma from '@/prisma/db';

const smtpConfig = {
  host: process.env.EMAIL_HOST,
  port: Number(process.env.EMAIL_PORT),
  secure: true, // use SSL
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  }
};


const brandColor = "#346df1"
const color = {
  background: "#f9f9f9",
  text: "#444",
  mainBackground: "#fff",
  buttonBackground: brandColor,
  buttonBorder: brandColor,
  buttonText: "#fff",
}

export const sendVerificationEmail = async (email: string, token: string) => {

  try {
    const transporter = nodemailer.createTransport(smtpConfig);


    const verificationUrl = `${process.env.NEXT_PUBLIC_BASE_URL}/register/verifyEmail?token=${token}`;

    await transporter.sendMail({
      from: process.env.EMAIL_USER,
      to: email,
      subject: 'Email Verification',
      html: `<body style="background: ${color.background};">
  <table width="100%" border="0" cellspacing="20" cellpadding="0"
    style="background: ${color.mainBackground}; max-width: 600px; margin: auto; border-radius: 10px;">
    <tr>
      <td align="center"
        style="padding: 10px 0px; font-size: 22px; font-family: Helvetica, Arial, sans-serif; color: ${color.text};">
        Sign in to <strong>E Commerce App</strong>
      </td>
    </tr>
    <tr>
      <td align="center" style="padding: 20px 0;">
        <table border="0" cellspacing="0" cellpadding="0">
          <tr>
            <td align="center" style="border-radius: 5px;" bgcolor="${color.buttonBackground}"><a href="${verificationUrl}"
                target="_blank"
                style="font-size: 18px; font-family: Helvetica, Arial, sans-serif; color: ${color.buttonText}; text-decoration: none; border-radius: 5px; padding: 10px 20px; border: 1px solid ${color.buttonBorder}; display: inline-block; font-weight: bold;">Sign
                in</a></td>
          </tr>
        </table>
      </td>
    </tr>
    <tr>
      <td align="center"
        style="padding: 0px 0px 10px 0px; font-size: 16px; line-height: 22px; font-family: Helvetica, Arial, sans-serif; color: ${color.text};">
        If you did not request this email you can safely ignore it.
      </td>
    </tr>
  </table>
</body>`,
    });
  } catch (error) {
    console.error(error);
  }

};

export default async function verifyEmail(tokenFromUrl: string) {

  try {
    const decryptedToken  = await decrypt(tokenFromUrl)

    const isEmailExist: object | null = await prisma.user.findUnique({
      where: {
        email: decryptedToken.email,
      },
    });


    if (isEmailExist) {
      console.log("email is already exist")
      return "email is already exist"
    }


    const decryptEmail = {
      email: decryptedToken.email
    }
    const emailToken = await encrypt(decryptEmail)
    console.log(decryptedToken + "           " + emailToken)

    const user = await prisma.user.create({
      data: {
        firstname: decryptedToken.firstname,
        middlename: decryptedToken.middlename,
        lastname: decryptedToken.lastname,
        email: decryptedToken.email,
        emailVerifyToken: emailToken,
        password: decryptedToken.password,
        roleId: decryptedToken.role,
      }
    })

    console.log(user)

    return "User is created"
  } catch (error) {
    console.error(error);
    return "token is expired"

  }

}

```

<p>You need to create a smtp server with gmail</p>
<p>Use this instruction
<a href="https://www.gmass.co/blog/gmail-smtp/">Gmail SMTP Settings: Easy Step-by-Step Setup Guide</a>
</p>


