# Authentication process in Students forum (Subject to be changed 🚧)

## User guideline

### Registration

In order to register on the platform you need to provide the following information

1. Your nickname that will be displayed to other users
2. Your email that will be used to confirm your registration, for further steps of registration process and verification of account as a "Confirmed student"
3. Your password
4. Profile picture
5. When you click the registration button you will get a confirmation email. By clicking on it you will be redirected back to the website and you will be logged in

### Login (currently does not include TOTP)

For login introduce the email, password and then in the OTP form write your TOTP from Google Authenticator or Microsoft Authenticator app

### Password reset (currently not implemented)

Press the _Forgot password_ link on the login page and then introduce the email of the account from which you lost the password. Access the link in the email and introduce the new password and confirm it. Use new credentials to login

## Developer guideline

Authentication on the platform is done using Supabase service. When signing up the user recieves in his email a redirect link to the website on the path `api/auth/confirm` that includes a token_hash.
This hash is sent back to supabase using the hadler that is attached to serve this route. It will read the token_hash, send it to supabase and recieve a valid cookie with following settings:

- HttpOnly - false. Set to this value because otherwise it is not possible to verify is a logged in user in client side components
- Secure - true
- SameSite - Lax

This cookie have encoded 2 jwt tokens. Refresh token and auth token. This cookie is used to validate all the user actions that require him to be loggen in.
