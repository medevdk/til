# Free email for custom domain

With Cloudflare use the free Email Routing feature that forwards email from `info@yourdomain.com` to personal email account (Gmail etc).

## Go to Cloudflare dashboard

- Select domain -> Email -> Email Routing
- Enable Email Routing -> Cloudflare will add the required record automatic
- Add:
  - Custom address: 'info'
  - Action: 'Send to an email'
  - Destination: personal Gmail or any address
- Confirm the verification email from Cloudflare

Test if it is working, send an email to info@mydomain.com (from any account except mydomain@gmail<a href=""></a>.com!) and check if it is received at myown@gmail.com

Now comes the tricky part.
If you want to reply to an email it must come from 'info@mydomain.com' and not from mydomain@gmail<a href=""></a>.com.
Therefore you need a Google 'app password'.

In Gmail go to 'manage your Google account' and Security.
Switch on 2-Step Verification.
Enter your phone number and confirm with the received code.
Next, to get an app password click <a href="https://myaccount.google.com/apppasswords">here</a>
Type any name (example gmail smtp). Copy the generated app password.
Go back to your Gmail ->; Settings (cog wheel) and 'See all settings&'.
Go 'Accounts and Import' -> 'Send mail as -> Add another email adress&rsquo;

- Name: Anything
- Email: info@mydomain.com

Next

- SMTP server: smtp.gmail.com
- Username: myown@gmail.com
- Password: Paste here the app password you just made

Add account and check your myown@gmail.com and confirm the link.
Done 🙂

Now send a new email to info@mydomain.com and when received (in mydomain@gmail<a href=""></a>.com) reply it.
Check the 'email from' in the replied email.
To send an email from any of the adresses:
In your gmail 'Compose' and in 'From' you can select which email adress to use.

Why like this?
You have now (max) 200 email addresses for free.
