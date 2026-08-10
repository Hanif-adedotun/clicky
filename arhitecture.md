# Clicky Onboarding Agent Architecture

## Overview
Clicky is an onboarding agent designed to help users complete tasks quickly inside web applications.
It works by mimicking cursor movements and guiding users step-by-step through in-app flows.

## Core Goal
The goal is to make onboarding simple for product teams:

1. Install the Clicky npm package.
2. Attach a relevant Markdown (`.md`) file describing task flows and guidance.
3. Bundle Clicky with your web application.

Once integrated, Clicky can act as a built-in assistant whenever a customer wants to achieve something quickly in the product.

## How It Helps End Users
- Provides clear, guided onboarding for key workflows.
- Mimics cursor movement to make instructions visual and easy to follow.
- Reduces confusion and time-to-value for new or returning users.

## Integration Model
- **Package-first setup:** Teams start by downloading and installing the npm package.
- **Markdown-driven guidance:** The attached Markdown file defines the onboarding instructions and contextual help.
- **Embedded assistant behavior:** Clicky runs inside the host application and can be invoked when users need fast guidance.

## Privacy and Data Handling
A core benefit of Clicky is privacy:

- All data remains local to the application environment.
- User onboarding and guidance data is not exposed externally.
- Teams retain full control over their guidance content and behavior.

## Intended Outcome
Clicky enables fast, repeatable, and privacy-preserving onboarding experiences that are easy to ship and maintain across web applications.