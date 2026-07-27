# Epic 1: Authentication & Session Management

The director can log in to the dashboard using email and password, and maintain a secure authenticated session with lockout protection.

**FRs covered:** FR-1, FR-2
**NFRs addressed:** NFR-6, NFR-7, NFR-8
**Stories:** 1.1, 1.2, 1.3, 1.4, 1.5

## Story 1.1: Set up project infrastructure

As a developer,
I want the .NET backend solution and React frontend project initialized with all required dependencies,
So that subsequent stories can build features on a stable foundation.

**Acceptance Criteria:**

**Given** no project exists
**When** the infrastructure story is complete
**Then** a .NET 10 solution exists with CineForge.Web, CineForge.Application, CineForge.Domain, and CineForge.Infrastructure projects
**And** a React 19 + Vite 6 project exists in `frontend/` with Tailwind CSS 4, TanStack Router/Table/Query, and shadcn/ui installed
**And** EF Core is configured with a connection to PostgreSQL (Neon)
**And** ASP.NET Core Identity is configured with cookie authentication
**And** the source tree matches the architecture spine specification

## Story 1.2: Implement director login with email and password

As a director,
I want to log in using my email address and password,
So that I can access the dashboard.

**Acceptance Criteria:**

**Given** the director is on the login page
**When** they enter a valid email and password and click "Sign in"
**Then** they are authenticated and redirected to the campaign list page
**And** a session cookie is set (httpOnly, secure, SameSite=Strict)
**Given** the director enters invalid credentials
**When** they click "Sign in"
**Then** they see a generic "Invalid email or password" error (no user enumeration)
**And** the login form remains on the page

## Story 1.3: Implement session management

As a director,
I want my authenticated session to persist across page reloads and expire after inactivity,
So that I don't have to log in repeatedly during a work session.

**Acceptance Criteria:**

**Given** the director has successfully logged in
**When** they navigate to any dashboard route
**Then** they remain authenticated without re-entering credentials
**Given** the director has been inactive for 24 hours
**When** they attempt to access a dashboard route
**Then** they are redirected to the login page with `?expired=true`
**Given** the director clicks "Logout"
**When** the logout action completes
**Then** the session cookie is cleared and they are redirected to `/login`
**Given** the director's session has expired
**When** they try to access a dashboard route
**Then** they are redirected to `/login?expired=true` and see "Your session has expired. Please log in again."

## Story 1.4: Implement account lockout after failed login attempts

As a director,
I want my account to be temporarily locked after repeated failed login attempts,
So that my account is protected from brute-force attacks.

**Acceptance Criteria:**

**Given** the director has entered incorrect credentials 5 times
**When** they attempt a 6th login
**Then** the account is locked for 15 minutes
**And** a countdown timer displays: "Too many failed attempts. Try again in {countdown}."
**And** the submit button is disabled until the lockout expires
**Given** the lockout period has expired
**When** the director attempts to log in
**Then** they can enter credentials normally
**Given** the director enters valid credentials after a lockout
**When** the lockout has expired
**Then** they are authenticated successfully

## Story 1.5: Implement login page UI with form validation and error states

As a director,
I want a clean, accessible login page with clear error messaging,
So that I can log in confidently without confusion.

**Acceptance Criteria:**

**Given** the director navigates to `/login`
**When** the page loads
**Then** a centered login form displays with email and password fields
**And** the form follows the design tokens (deep slate primary, warm amber accent, Inter font)
**Given** the director types in the email field
**When** the email field is empty
**Then** the "Sign in" button is disabled
**Given** the director submits with invalid credentials
**When** the server responds with an error
**Then** a destructive toast displays: "Invalid email or password."
**And** the error is announced via `aria-live`
**Given** the director uses keyboard navigation
**When** they tab through the form
**Then** focus rings use the accent color at AA contrast
**And** all interactive elements have visible focus states
