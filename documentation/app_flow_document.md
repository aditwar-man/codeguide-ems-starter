# App Flow Document for Educational Management System Starter (codeguide-ems-starter)

## Onboarding and Sign-In/Sign-Up
When a new user arrives at the application, they first see a clean landing page with options to sign in or sign up. The sign-up page welcomes them with input fields for email, password, and a confirmation of the password. After filling these in, the user clicks the sign-up button, which calls the Better Auth sign-up endpoint. A verification email may be sent, depending on the configuration, and once verified, the user is redirected back to the sign-in page.

On the sign-in page, the user enters their email and password and submits the form. The app calls the Better Auth sign-in endpoint, which validates credentials. When successful, a session cookie is created and the user is redirected to their role-specific dashboard. If the user forgets their password, they click the "Forgot Password" link, which asks for their email address. The system sends a password reset link, and when the user clicks that link, they land on a page to enter a new password. Submitting the new password calls the password reset endpoint, and on success the user is prompted to sign in again with the new credentials.

Social login is not enabled by default in this starter, but it could be added by extending the Better Auth configuration with OAuth providers. Signing out is done by clicking a "Log Out" button in the navigation, which calls the sign-out endpoint and clears the user session, then redirects to the landing page.

## Main Dashboard or Home Page
Once signed in, the user lands on a shared dashboard layout. At the top is a header showing the app logo, the user’s name, and a log-out button. On the left side is a sidebar navigation that adapts to the user’s role: for a `Kurikulum` user it shows links for class management, curriculum planning, and question bank; for a `Guru` user it shows links for grade entry, attendance, and lesson resources; for a `Siswa` user it shows links for viewing grades, schedule, and study materials.

In the main area of the dashboard, widgets appear. A `Kurikulum` user sees high-level charts of grade distributions and pending teacher assignments. A `Guru` user sees a list of their classes with quick links to grade input tables. A `Siswa` user sees their upcoming classes and recent grades in a simple table. Each widget links to a full page of details when clicked. The layout is responsive and uses Tailwind CSS, so it adjusts on smaller screens.

Users can click on any sidebar link and the app will transition to the corresponding page under the `/app` route. The URL changes to reflect the role and page, for example `/guru/grades` or `/siswa/schedule`.

## Detailed Feature Flows and Page Transitions
When a `Guru` clicks the Grades link, they arrive on a page showing a data table with columns for student names, assignments, and grade inputs. At the top of the page is a class selector dropdown. Choosing a class triggers a new server request that fetches students and assignments for that class. The table updates accordingly. The teacher can enter or edit grades in the cells and then click a "Save" button. Clicking Save calls the API route under `/api/teacher/grades`, which validates the entries and writes them to the PostgreSQL database. On success, the page shows a green success toast at the top.

If a `Kurikulum` user wants to manage the question bank, they click the Bank Soal link. They see a paginated list of question sets, each with edit and delete icons. Clicking edit opens a form that shows existing questions in a repeater component. The user can modify text, add new questions, or remove old ones. When they submit, the app calls `/api/admin/bank-soal/update` and then reloads the list view.

For file uploads, such as lesson plans, users click the Upload Resources link. A form appears with a file input. Selecting a file triggers a request to an API route that returns a presigned URL from AWS S3. The frontend then uploads the file directly to S3. Once complete, the S3 URL is saved in the database through another API call, and the page lists the newly uploaded document.

When a `Siswa` clicks Schedule, the page fetches their class schedule via `/api/student/schedule`. It renders a calendar with time slots and subjects. Clicking a time slot provides more details in a modal, such as teacher name and classroom link.

Real-time chat is available for consultations. The user navigates to the Chat page under their portal. A Socket.IO connection is established to the custom Node.js server. Messages sent by the user appear in the chat window immediately and propagate to the teacher’s view. Incoming messages show up in real time. If the connection is lost, a banner appears at the top indicating offline status and the chat attempts to reconnect automatically.

All these pages share the same sidebar and header. Clicking the logo in the header always brings the user back to their main dashboard.

## Settings and Account Management
Every user can access their profile settings by clicking their avatar or name in the header and choosing the Profile link. The Profile page shows their name, email, and role. The user can update their name or email and click Save to call `/api/user/update`. To change the password, there is a separate form on the same page that asks for the current password and the new password twice. Submitting that form calls `/api/user/change-password`. Notifications settings appear below, with toggles for email alerts on new messages or upcoming deadlines. Toggling these options and saving updates the user preferences in the database.

If a user has subscriptions or billing plans, there is a Billing link in the sidebar. On the Billing page, users can view their current plan and payment method. They can update card details via an embedded payment form that communicates with Stripe. When changes are saved, the page shows a confirmation that the billing update was successful. From Billing or Profile, users can click back to Dashboard using a link at the top.

## Error States and Alternate Paths
If a user enters invalid credentials at sign-in, an inline error message appears in red under the form fields instructing them to check their email and password. On sign-up, if a password confirmation does not match, an error is shown next to the confirmation field.

In any form that calls an API, if the server returns a validation error, the page highlights the offending field and shows the error text. If the server is unreachable due to network issues, a notification bar appears at the top saying "Network error, please check your connection." The app will retry or allow the user to refresh.

When a user attempts to access a page they are not authorized for, such as a `Siswa` typing `/guru/grades` in the URL bar, the middleware denies access and redirects to a 403 Forbidden page that explains they do not have permission. If a user navigates to a non-existent route, they see a friendly 404 page with a link to return to their dashboard.

During critical failures, such as database downtime, the app shows a maintenance page that asks the user to try again later.

## Conclusion and Overall App Journey
A new user visits the app, signs up or signs in, and lands on their tailored dashboard. From there, they use the sidebar to navigate to core features like managing grades, reviewing schedules, uploading resources, or chatting. They access settings to update their profile or billing information, and when errors occur the app gently guides them back on track. The flow moves smoothly from page to page under the Next.js App Router, with secure server-side data fetching and clear client-side feedback. Over time, educators and students alike complete daily tasks such as entering grades, checking assignments, and collaborating through chat, all within this cohesive Educational Management System starter template.