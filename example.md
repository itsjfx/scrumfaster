# my cool board

## Sprint 1

* [ ] Profile avatars
    * [ ] Create database migration for avatar field [@itsjfx] [status=Done] [labels=database] [1]
        * Name the field `avatar` in the `users` table
        * Set value for existing users to https://...
    * [ ] Accept avatar parameter in `update_user` API call [@itsjfx] [labels=api] [1]
        * Use existing image upload mechanisms
        * Limit image size to 10mb
    * [ ] Display and allow updating avatars on frontend [labels=frontend] [2]
        * Only display avatars on the users public profile page
        * Thumbnails aside comments to be implemented in later card
* [ ] Dark mode
    * [ ] Add ui toggle for dark mode [labels=frontend] [1]
        * Store preference in local storage
    * [ ] Implement styles [labels=frontend] [2]
        * Apply styles dynamically based on user preference
* [ ] Delete jeff from database [labels=database] [1]
