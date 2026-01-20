# MFA Authentication

Minimal Express + Passport + TOTP (speakeasy) example.

## Prerequisites
- Node.js (16+)
- MongoDB (Atlas or local)
- Copy [.env-sample](.env-sample) -> `.env` and set values.

Files of interest:
- [package.json](package.json)
- [.env-sample](.env-sample)
- [src/index.js](src/index.js)
- [src/config/dbConnect.js](src/config/dbConnect.js)
- [src/config/passportConfig.js](src/config/passportConfig.js)
- [src/routes/authRoutes.js](src/routes/authRoutes.js)
- [src/controllers/authController.js](src/controllers/authController.js) — exports: [`register`](src/controllers/authController.js), [`login`](src/controllers/authController.js), [`authStatus`](src/controllers/authController.js), [`logout`](src/controllers/authController.js), [`setup2FA`](src/controllers/authController.js), [`verify2FA`](src/controllers/authController.js), [`reset2FA`](src/controllers/authController.js)
- [src/models/user.js](src/models/user.js)

## Install
```sh
npm install
```

## Environment
Copy [.env-sample](.env-sample) to `.env` and set:
- PORT
- SESSION_SECRET
- JWT_SECRET
- CONNECTION_STRING

## Run (development)
```sh
npm run dev
```
Server entry: [src/index.js](src/index.js). Default port is from `PORT` or 7002.

## API (base URL: http://localhost:$PORT/api/auth)
- POST /register — create account (`register` in [src/controllers/authController.js](src/controllers/authController.js))
  - body: { "username": "user", "password": "pass" }
- POST /login — session login via Passport Local (`login` in [src/controllers/authController.js](src/controllers/authController.js))
  - body: { "username": "user", "password": "pass" }
  - Note: uses cookie-based session; preserve cookies between requests.
- GET /status — check session (`authStatus` in [src/controllers/authController.js](src/controllers/authController.js))
- POST /logout — logout (`logout` in [src/controllers/authController.js](src/controllers/authController.js))

2FA (requires authenticated session):
- POST /2fa/setup — enable 2FA, returns QR and secret (`setup2FA` in [src/controllers/authController.js](src/controllers/authController.js))
- POST /2fa/verify — verify TOTP and receive JWT (`verify2FA` in [src/controllers/authController.js](src/controllers/authController.js))
  - body: { "token": "<6-digit-code>" }
- POST /2fa/reset — disable 2FA (`reset2FA` in [src/controllers/authController.js](src/controllers/authController.js))

## Quick curl examples
Register:
```sh
curl -X POST http://localhost:7002/api/auth/register -H "Content-Type: application/json" -d '{"username":"u","password":"p"}'
```
Login (save cookies):
```sh
curl -c cookiejar -X POST http://localhost:7002/api/auth/login -H "Content-Type: application/json" -d '{"username":"u","password":"p"}'
```
Setup 2FA (use saved cookies):
```sh
curl -b cookiejar -X POST http://localhost:7002/api/auth/2fa/setup
```
Verify 2FA:
```sh
curl -b cookiejar -X POST http://localhost:7002/api/auth/2fa/verify -H "Content-Type: application/json" -d '{"token":"123456"}'
```

## Notes
- Sessions handled via `express-session` (see [src/index.js](src/index.js) and [.env-sample](.env-sample)).
- User model: [src/models/user.js](src/models/user.js).
- Passport local strategy and session serialization: [src/config/passportConfig.js](src/config/passportConfig.js).
- JWT for 2FA issued with `JWT_SECRET`.
