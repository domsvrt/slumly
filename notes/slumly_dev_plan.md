# 📓 SLUMLY — Development Plan
> *"Fold. Pass. Connect."*

---

## 📌 Table of Contents
1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Database Schema](#3-database-schema)
4. [Feature Breakdown](#4-feature-breakdown)
5. [Routes](#5-routes)
6. [Controllers & Services](#6-controllers--services)
7. [UI/UX Design Guidelines](#7-uiux-design-guidelines)
8. [Development Phases](#8-development-phases)
9. [Folder Structure Conventions](#9-folder-structure-conventions)
10. [Testing Plan](#10-testing-plan)

---

## 1. Project Overview

| Field       | Details                                      |
|-------------|----------------------------------------------|
| **Name**    | Slumly                                       |
| **Tagline** | *"Fold. Pass. Connect."*                     |
| **Type**    | Social Media Web Application                 |
| **Stack**   | Laravel (Backend) + Vite + Bootstrap (Frontend) |
| **Concept** | A modern slambook-inspired platform where users pass "Slum Notes" to each other — publicly, privately, or anonymously — and reply like a comment thread. |

---

## 2. Tech Stack

| Layer         | Technology & Version                                 |
|---------------|------------------------------------------------------|
| Backend       | Laravel ^13.0 (laravel/framework)                    |
| PHP           | ^8.4                                                 |
| Frontend      | Blade Templates + Bootstrap 5 + Vite ^8.0.0          |
| CSS Framework | Tailwind CSS ^4.0.7                                  |
| Database      | PostgreSQL                                           |
| Auth          | Laravel Fortify ^1.34                                |
| File Storage  | Laravel Storage (local / S3)                         |
| Real-time     | Laravel Echo + Pusher                                |
| Testing       | PHPUnit ^12, Pest ^4.6                               |
| Queue         | Laravel Queue                                        |

---

## Tool Versions

| Tool/Library                  | Version/Constraint      |
|-------------------------------|------------------------|
| PHP                           | ^8.4                   |
| laravel/framework             | ^13.0                  |
| laravel/fortify               | ^1.34                  |
| laravel/tinker                | ^3.0                   |
| livewire/livewire             | ^4.1                   |
| livewire/flux                 | ^2.13.1                |
| laravel/boost                 | ^2.2                   |
| laravel/pail                  | ^1.2.5                 |
| laravel/pint                  | ^1.27                  |
| laravel/sail                  | ^1.53                  |
| pestphp/pest                  | ^4.6                   |
| pestphp/pest-plugin-laravel   | ^4.1                   |
| phpunit/phpunit               | ^12                    |
| tailwindcss                   | ^4.0.7                 |
| vite                          | ^8.0.0                 |
| laravel-vite-plugin           | ^3.0.0                 |
| @tailwindcss/vite             | ^4.1.11                |
| autoprefixer                  | ^10.4.20               |
| concurrently                  | ^9.0.1                 |
---

## 3. Database Schema

### `users`
| Column         | Type         | Notes                        |
|----------------|--------------|------------------------------|
| id             | bigint PK    |                              |
| username       | string       | unique, public handle        |
| name           | string       |                              |
| email          | string       | unique                       |
| password       | string       |                              |
| avatar         | string/null  | profile photo path           |
| bio            | text/null    | short user description       |
| is_private     | boolean      | default false                |
| allow_anon_notes | boolean    | allow anonymous senders      |
| created_at     | timestamp    |                              |
| updated_at     | timestamp    |                              |

---

### `friendships`
| Column       | Type      | Notes                                          |
|--------------|-----------|------------------------------------------------|
| id           | bigint PK |                                                |
| requester_id | bigint FK | user who sent the friend request               |
| receiver_id  | bigint FK | user who received the request                  |
| status       | enum      | `pending`, `accepted`, `blocked`               |
| created_at   | timestamp |                                                |
| updated_at   | timestamp |                                                |

---

### `slum_notes`
| Column        | Type      | Notes                                             |
|---------------|-----------|---------------------------------------------------|
| id            | bigint PK |                                                   |
| sender_id     | bigint FK | nullable (null = anonymous)                       |
| receiver_id   | bigint FK | the user who receives the note                    |
| content       | text      | the note body                                     |
| is_anonymous  | boolean   | hides sender identity from receiver               |
| visibility    | enum      | `public`, `friends`, `specific`                   |
| is_folded     | boolean   | default true (note starts "folded"/unread)        |
| color         | string    | note paper color (for UI theming)                 |
| sticker       | string/null | optional sticker/emoji decoration               |
| created_at    | timestamp |                                                   |
| updated_at    | timestamp |                                                   |

> **Note:** When `visibility = specific`, a pivot table `slum_note_recipients` maps which specific users can see the note.

---

### `slum_note_recipients` *(for specific visibility)*
| Column      | Type      | Notes                      |
|-------------|-----------|----------------------------|
| id          | bigint PK |                            |
| note_id     | bigint FK | references slum_notes.id   |
| user_id     | bigint FK | specific recipient user id |

---

### `note_replies`
| Column       | Type      | Notes                                       |
|--------------|-----------|---------------------------------------------|
| id           | bigint PK |                                             |
| note_id      | bigint FK | references slum_notes.id                    |
| user_id      | bigint FK | nullable (null = anonymous)                 |
| content      | text      | reply body                                  |
| is_anonymous | boolean   | hides replier identity                      |
| created_at   | timestamp |                                             |
| updated_at   | timestamp |                                             |

---

### `notifications`
> Use Laravel's built-in `notifications` table via `php artisan notifications:table`

---

## 4. Feature Breakdown

### 4.1 Authentication
- [x] Register with username, name, email, password
- [x] Login / Logout
- [x] Forgot password / Reset password
- [x] Email verification

---

### 4.2 User Profile
- [ ] View own profile (`/profile`)
- [ ] View other user profiles (`/@{username}`)
- [ ] Edit profile (avatar, bio, name, settings)
- [ ] Toggle: allow anonymous notes
- [ ] Toggle: private profile
- [ ] Profile displays:
  - Notes **sent** by user (public ones)
  - Notes **received** by user (public ones)
  - Friends list / count

---

### 4.3 Slum Notes (Core Feature)

#### Sending a Note
- [ ] Compose a note with:
  - **Content** (text, max 500 chars)
  - **Recipient** (search by username)
  - **Visibility**: `public` | `friends only` | `specific user(s)`
  - **Anonymous toggle** (hide your identity)
  - **Color picker** (note paper color)
  - **Sticker picker** (optional decoration)
- [ ] Notes start as "folded" (receiver must "unfold" to read)
- [ ] Sender sees sent notes on their profile

#### Receiving a Note
- [ ] Inbox page shows all received notes (folded state by default)
- [ ] Click/tap to **"unfold"** a note (animated unfold interaction)
- [ ] After unfolding, receiver can **reply**

#### Note Visibility Rules
| Visibility   | Who Can See It                        |
|--------------|---------------------------------------|
| `public`     | Everyone (on receiver's profile feed) |
| `friends`    | Sender's and receiver's mutual friends|
| `specific`   | Only the chosen user(s)               |

---

### 4.4 Replies (Comment Feature)
- [ ] Receiver and other allowed users can reply to a note
- [ ] Replies can also be **anonymous or named**
- [ ] Replies are shown in a thread under the unfolded note
- [ ] Nested replies: max 1 level deep (reply to the original note only)
- [ ] Sender is notified when their note gets a reply

---

### 4.5 Friends System
- [ ] Send / accept / decline friend requests
- [ ] Block a user
- [ ] Friends list page
- [ ] Suggestions based on mutual friends

---

### 4.6 Notifications
- [ ] New note received
- [ ] Someone replied to your note
- [ ] Friend request received / accepted
- [ ] Real-time bell icon (Laravel Echo + Pusher)

---

### 4.7 Feed / Home Page
- [ ] Shows a feed of **public notes** from friends or everyone
- [ ] Notes displayed as folded paper cards
- [ ] Infinite scroll or pagination

---

### 4.8 Search
- [ ] Search users by username or name
- [ ] Search public notes by content keyword

---

## 5. Routes

```php
// Auth
Route::get('/login', [AuthController::class, 'showLogin']);
Route::post('/login', [AuthController::class, 'login']);
Route::post('/logout', [AuthController::class, 'logout']);
Route::get('/register', [AuthController::class, 'showRegister']);
Route::post('/register', [AuthController::class, 'register']);

// Home / Feed
Route::get('/', [FeedController::class, 'index'])->middleware('auth');

// Profile
Route::get('/profile', [ProfileController::class, 'ownProfile'])->middleware('auth');
Route::get('/@{username}', [ProfileController::class, 'show']);
Route::put('/profile', [ProfileController::class, 'update'])->middleware('auth');

// Slum Notes
Route::middleware('auth')->group(function () {
    Route::get('/notes', [NoteController::class, 'inbox']);           // Inbox
    Route::get('/notes/sent', [NoteController::class, 'sent']);       // Sent notes
    Route::post('/notes', [NoteController::class, 'store']);          // Send a note
    Route::get('/notes/{id}', [NoteController::class, 'show']);       // View/unfold a note
    Route::delete('/notes/{id}', [NoteController::class, 'destroy']); // Delete a note
    Route::post('/notes/{id}/unfold', [NoteController::class, 'unfold']); // Mark as unfolded
});

// Replies
Route::middleware('auth')->group(function () {
    Route::post('/notes/{id}/replies', [ReplyController::class, 'store']);
    Route::delete('/replies/{id}', [ReplyController::class, 'destroy']);
});

// Friends
Route::middleware('auth')->group(function () {
    Route::get('/friends', [FriendController::class, 'index']);
    Route::post('/friends/{userId}', [FriendController::class, 'sendRequest']);
    Route::put('/friends/{userId}/accept', [FriendController::class, 'accept']);
    Route::put('/friends/{userId}/decline', [FriendController::class, 'decline']);
    Route::delete('/friends/{userId}', [FriendController::class, 'remove']);
    Route::post('/friends/{userId}/block', [FriendController::class, 'block']);
});

// Notifications
Route::get('/notifications', [NotificationController::class, 'index'])->middleware('auth');
Route::post('/notifications/read-all', [NotificationController::class, 'readAll'])->middleware('auth');

// Search
Route::get('/search', [SearchController::class, 'index']);
```

---

## 6. Controllers & Services

### Controllers (in `app/Http/Controllers/`)
```
AuthController.php
FeedController.php
ProfileController.php
NoteController.php
ReplyController.php
FriendController.php
NotificationController.php
SearchController.php
```

### Services (in `app/Services/`)
```
NoteService.php         — Business logic for creating, sending, visibility checks
FriendService.php       — Friendship state management
NotificationService.php — Dispatching and reading notifications
AnonymityService.php    — Resolving sender identity based on anonymous flag
```

### Models (in `app/Models/`)
```
User.php
SlumNote.php
NoteReply.php
Friendship.php
```

### Observers
```
NoteObserver.php  — Fires notification when note is received
ReplyObserver.php — Fires notification when a reply is posted
```

---

## 7. UI/UX Design Guidelines

### Design Theme: *Modern Slambook*

| Element           | Style                                                    |
|-------------------|----------------------------------------------------------|
| Font (Headings)    | `Playfair Display` or `Abril Fatface` — classic notebook feel |
| Font (Body)        | `Nunito` or `DM Sans` — clean, readable                 |
| Primary Color      | Warm cream / off-white `#FDF6EC`                        |
| Accent Color       | Deep rose `#C0392B` or indigo `#4A3F6B`                 |
| Note Card Colors   | Pastel: pink, yellow, mint, lavender, peach             |
| Borders            | Dashed/dotted to mimic notebook paper                   |
| Backgrounds        | Subtle lined-paper or grid texture                      |
| Buttons            | Rounded, slightly stamped feel                          |
| Animations         | CSS paper-unfold transition when opening a note         |

### Key UI Components
- **Note Card** — Folded paper card with color + sticker, click to unfold with animation
- **Compose Modal** — Full-page modal styled like a blank notebook page
- **Profile Page** — Looks like an open slambook with tabbed sections
- **Inbox** — Stack of folded notes, like a pile of passed notes on a desk
- **Feed** — Scrollable wall of pinned/folded note cards

---

## 8. Development Phases

### ✅ Phase 1 — Foundation (Week 1–2)
- [ ] Laravel project setup, `.env` config, DB migration
- [ ] Authentication (Breeze)
- [ ] User model + profile setup
- [ ] Basic Blade layout with Bootstrap 5 + Vite
- [ ] Global design system (colors, fonts, note card component)

### 📝 Phase 2 — Core Notes Feature (Week 3–4)
- [ ] SlumNote model + migration
- [ ] Compose & send note (with visibility + anonymous toggle)
- [ ] Inbox (received notes, folded state)
- [ ] Unfold animation + note view
- [ ] Sent notes on profile

### 💬 Phase 3 — Replies & Social (Week 5)
- [ ] NoteReply model + migration
- [ ] Reply thread on unfolded note
- [ ] Anonymous reply toggle
- [ ] Friends system (request, accept, block)

### 🔔 Phase 4 — Notifications & Feed (Week 6)
- [ ] Notification system (DB + real-time with Pusher)
- [ ] Home feed (public notes from friends/global)
- [ ] Search (users + notes)

### 🎨 Phase 5 — Polish & Testing (Week 7–8)
- [ ] Full UI polish (slambook aesthetic)
- [ ] Mobile responsiveness
- [ ] PHPUnit / Pest feature tests
- [ ] Performance optimization (eager loading, pagination)
- [ ] Security audit (anonymous identity leak check, visibility enforcement)

---

## 9. Folder Structure Conventions

```
app/
├── Http/
│   ├── Controllers/
│   ├── Requests/          ← Form validation (StoreNoteRequest, etc.)
│   └── Middleware/
├── Models/
├── Services/
├── Observers/
└── Notifications/

resources/
├── views/
│   ├── layouts/
│   │   └── app.blade.php
│   ├── auth/
│   ├── feed/
│   ├── notes/
│   │   ├── index.blade.php    ← Inbox
│   │   ├── show.blade.php     ← Unfolded note view
│   │   └── compose.blade.php  ← Send a note
│   ├── profile/
│   └── components/
│       ├── note-card.blade.php
│       ├── reply-thread.blade.php
│       └── friend-card.blade.php
├── css/
│   └── app.css
└── js/
    └── app.js

database/
├── migrations/
└── seeders/
    ├── UserSeeder.php
    └── NoteSeeder.php

routes/
└── web.php
```

---

## 10. Testing Plan

### Feature Tests (PHPUnit / Pest)
| Test                              | Scenario                                        |
|-----------------------------------|-------------------------------------------------|
| `NoteVisibilityTest`              | Public note visible to all; friends-only note hidden from non-friends |
| `AnonymousSenderTest`             | Sender identity not exposed when anonymous flag is true |
| `NoteReplyTest`                   | Only authorized users can reply to a note       |
| `FriendshipTest`                  | Correct state transitions (pending → accepted)  |
| `NotificationTest`                | Notification fired on note receipt and reply    |
| `ProfileNoteDisplayTest`          | Profile shows only public sent notes            |

### Manual QA Checklist
- [ ] Send note anonymously and verify sender is hidden on receiver's end
- [ ] Send a `friends-only` note and verify non-friends cannot see it
- [ ] Unfold animation works on mobile and desktop
- [ ] Reply thread shows anonymous/named replies correctly
- [ ] Block user prevents note sending in both directions
- [ ] Notifications arrive in real-time without page refresh

---

> 📌 **Next Step:** Start with `Phase 1`. Run `php artisan migrate`, set up Breeze auth, and scaffold the global layout with the slambook design system.

---
*Slumly Dev Team — Internal Document*
