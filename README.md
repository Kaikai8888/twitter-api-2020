# Simple Twitter API

A RESTful backend for a simplified Twitter clone, built with **Node.js, Express, Sequelize and MySQL**. It covers JWT authentication, tweets / replies / likes, followships, an admin console, and **real-time public & private chat rooms powered by Socket.IO**.

Co-developed by two backend engineers ([Kaikai8888](https://github.com/Kaikai8888) & [ccyang02](https://github.com/ccyang02)) and integrated with a separate frontend team through a shared Swagger (OpenAPI 3) spec.

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [API Overview](#api-overview)
- [Real-time Chat](#real-time-chat)
- [Data Model](#data-model)
- [Getting Started](#getting-started)
- [Testing](#testing)
- [Contributors](#contributors)

## Features

| Area | Features |
| --- | --- |
| Auth | Sign up, sign in with JWT; separate permission guards for users and admins |
| User | Profile, tweets, replies, liked tweets, followers / followings, user list, profile update with avatar & cover upload (Imgur) |
| Tweet | List / view / post tweets, reply, like, unlike |
| Followship | Follow and unfollow users |
| Admin | Admin sign-in, list & delete any tweet, list all users with their social stats |
| Chat | Real-time public chat room, 1-on-1 private chat, online user list, unread message notifications |

## Tech Stack

- **Runtime / Framework:** Node.js 10, Express 4
- **Database / ORM:** MySQL, Sequelize 4 (migrations & seeders)
- **Auth:** Passport (local + JWT), bcryptjs
- **Real-time:** Socket.IO 3
- **Validation:** express-validator
- **File upload:** multer + Imgur API
- **Testing:** Mocha, Chai, Sinon, Supertest
- **Docs:** Swagger (OpenAPI 3)

## API Overview

Full API documentation: [SwaggerHub](https://app.swaggerhub.com/apis-docs/ccyang02/SimpleTwitter/1.1.1)

All endpoints are prefixed with `/api`. Apart from sign-up and sign-in, every endpoint requires `Authorization: Bearer <token>`.

| Method | Endpoint | Description |
| --- | --- | --- |
| POST | `/users` | Sign up |
| POST | `/signin` | User sign in |
| GET | `/users/currentUser` | Get the current user |
| GET | `/users` | Get users |
| GET | `/users/:id` | Get a user profile |
| PUT | `/users/:id` | Update a user profile |
| GET | `/users/:id/tweets` | Get a user's tweets |
| GET | `/users/:id/replied_tweets` | Get a user's replies |
| GET | `/users/:id/likes` | Get tweets a user liked |
| GET | `/users/:id/followers` | Get a user's followers |
| GET | `/users/:id/followings` | Get a user's followings |
| POST | `/followships` | Follow a user |
| DELETE | `/followships/:followingId` | Unfollow a user |
| GET | `/tweets` | Get all tweets |
| POST | `/tweets` | Post a tweet |
| GET | `/tweets/:id` | Get a tweet |
| GET | `/tweets/:id/replies` | Get replies of a tweet |
| POST | `/tweets/:id/replies` | Reply to a tweet |
| POST | `/tweets/:id/like` | Like a tweet |
| POST | `/tweets/:id/unlike` | Unlike a tweet |
| POST | `/admin/signin` | Admin sign in |
| GET | `/admin/tweets` | Get all tweets (admin) |
| DELETE | `/admin/tweets/:id` | Delete a tweet (admin) |
| GET | `/admin/users` | Get all users (admin) |

## Real-time Chat

- Socket connections are authenticated with the same JWT through a Passport middleware, and admins are rejected.
- **Public room:** everyone joins on connect, loads chat history, and sees an online user list that updates live.
- **Private rooms:** each pair of users gets its own channel. Messages are stored in the database.
- **Unread notifications:** a `Read` table records when each user last read each channel, which drives the unread counts for both public and private messages.
- A server-rendered (Handlebars) chat UI is served at `/chat` for testing.

## Data Model

```
User ──< Tweet ──< Reply
 │         └──< Like >── User
 ├──< Followship >── User   (follower / following)
 ├──< Chatpublic            (public room messages)
 ├──< Channel ──< Chatprivate   (1-on-1 rooms & messages)
 └──< Read                  (last-read time per channel)
```

## Getting Started

### Prerequisites

- Node.js 10+
- MySQL

### Installation

```bash
git clone https://github.com/Kaikai8888/twitter-api-2020.git
cd twitter-api-2020
npm install
```

### Environment Variables

Create a `.env` file from `.env.example`:

```bash
cp .env.example .env
```

| Variable | Description |
| --- | --- |
| `JWT_SECRET` | Secret used to sign JWTs |
| `PORT` | Server port (default `3000`) |
| `IMGUR_CLIENT_ID` | Imgur client ID for image upload ([apply here](https://api.imgur.com/oauth2/addclient)) |
| `DOMAIN` | Server domain used by the chat pages, e.g. `http://localhost:3000/` |

### Database

Update the database settings in `config/config.json`, then run:

```bash
npx sequelize db:migrate
npx sequelize db:seed:all

# undo all migrations if needed
npx sequelize db:migrate:undo:all
```

### Run

```bash
npm run dev
```

The server runs on `http://localhost:3000`. You can try the APIs with [Postman](https://www.postman.com/), starting with `POST /api/signin`.

> **Note:** The npm scripts use Windows `set` syntax. On macOS / Linux, run `NODE_ENV=development npx nodemon app.js` instead.

### Seed Accounts

| Role | Account | Email | Password |
| --- | --- | --- | --- |
| Admin | `root` | root@example.com | `12345678` |
| User | `User1` ~ `User5` | user1@example.com ~ user5@example.com | `12345678` |

## Testing

```bash
# Windows
npm test

# macOS / Linux
NODE_ENV=test npx mocha test/*/*.js --exit --recursive --timeout 5000
```

Tests cover the models (User, Tweet, Reply, Like, Followship) and the request flows for users, tweets, replies, likes, followships and admin.

## Contributors

- [Kaikai8888](https://github.com/Kaikai8888): data models & seeders, JWT auth & permission middleware, User / Followship APIs, Imgur upload, SQL injection hardening, Socket.IO auth & chat foundation, chat room UI
- [ccyang02](https://github.com/ccyang02): Tweet / Admin APIs, validators, chat message persistence & history, private chat flow, online users & unread notifications, Heroku deployment
