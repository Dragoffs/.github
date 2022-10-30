# Table of Contents

[SL2](#SL2)

[SL2 Database Documentation](#SL2-Database-Documentation)

[SL2 Client](#SL2-Client)

[Logfiles and Parsing](#Logfiles-and-Parsing)

---

# SL2

## Folder contents

1. **backend**

This folder contains the API code.
- Express.js App http://localhost:8000
- MongoDB Integration
- Swagger UI Documentation

2. **client**

This folder contains the frontend code.
- Next.js App http://localhost:3000/

## Requirements

1. [Git](https://git-scm.com/)
2. [Node.js](https://nodejs.org/en/) _(v.16.16.0 or latest)_

## How to run

### Linux 

1. Clone this repo `git clone https://github.com/Dragoffs/SL2.git`
2. Move into the directory:`cd SL2`
3. Run `npm start`
4. Open `localhost:3000` and you should be able to see the app.

### Windows

1. Clone this repo `git clone https://github.com/Dragoffs/SL2.git`
2. Move into the directory SL2: `cd SL2`
3. Run `npm install`
4. Run `npm run windows`
5. Open `localhost:3000` and you should be able to see the app.

---

# SL2 Database Documentation

## MongoDB 
For our SL2 product, we are utilizing MangoDB to store our parsed log data, student/professor information, and classes/courses information.
Our database is hosted on Mongo Atlas, a cloud-based database hosting service that has free options for storing information. 
The SL2 cluster is currently owned by Nathan Radin, Database Admin, but ownership can be moved to another admin when handoff occurs.


### Mongo Collection/Structure
The SL2 database contains three separate collections for data storing:
- Classes
- Professors
- Students

The below sections will provide information on how are collections are set up. We have used the following formatting for consistency:

Ex. - Readable Name (*Api Name*) - *Data structure* **brief explantation.**

#### Classes
The classes collection will contain information directly associated with the classes being covered for the product.
As of our Beta version on 10/18/2022, each class has the following datapoints:

- Class Name (*class_name*): *String* **English title of class.**
- Class Code (*class_code*): *String* **Class code associated with title.**

#### Professors
The professors collection will contain information about each professor as an individual. 
As of our Beta version on 10/18/2022, each professor has the following datapoints:

- First Name (*first_name*): *String* **First name of the professor.**
- Last Name (*last_name*): *String* **Last name of the professor.**
- Courses (*courses*): *Array* **List of courses professor is teaching.**
- Email (*email*): *String* **Email associated with professor.**
- Password (*password*): *String* **Password professor uses to log into SL2.**
- Pinned Students (*pinned*): *Array* **List of pinned students, used for quick reference on SL2 home page.**

#### Students
The students collection will contain all information about students, as well as their respective parsed logs.
As of our Beta version on 10/18/2022, each student has the following datapoints:

- University ID (*uid*): *Integar* **Id given to student who attends RIT.**
- Username (*username*): *String* **Username of student, given by RIT. Ex. abc1234.**
- First Name (*first_name*): *String* **First name of student.**
- Last Name (*last_name*): *String* **Last name of student.**
- Courses (*courses*): *Array* **List of courses student is taking.**
- Email (*email*): *String* **RIT email of student.**
- IP Address (*ip*): *String* **IP Address of the student.**
- Parsed Login Logs (*logs*): *Array* **List of Log objects pertaining to each login attempt by user.**
    - Datetime (*datetime*): *Datetime* **Date and time of that particular login attempt.**
    - Result (*result*): *Strin*g **Result of the connected log file, either Success, Disconnect, or Failure.**

---

# SL2 Client

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `pages/index.js`. The page auto-updates as you edit the file.

[API routes](https://nextjs.org/docs/api-routes/introduction) can be accessed on [http://localhost:3000/api/hello](http://localhost:3000/api/hello). This endpoint can be edited in `pages/api/hello.js`.

The `pages/api` directory is mapped to `/api/*`. Files in this directory are treated as [API routes](https://nextjs.org/docs/api-routes/introduction) instead of React pages.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.

---

# Logfiles and Parsing

[Repository to store dummy data and log parsing utilities](https://github.com/Dragoffs/Data)

## Logging on Solace

`rsyslogd` is the logging utility currently in use on Solace.

`rsyslogd` logs a stored under `/var/log/` in CentOS (the OS used by Solace).

The following format is used for each of the failure and success modes of a login / file copy over `ssh`:

#### Success: correct password

```
<timestamp (m d hh:mm:ss)> <machine name> <system daemon (sshd)>: Accepted password for <user> from <ip address> port <port number> <ssh protocol (ssh2)>
```

#### Failure: wrong password

```
<timestamp (m d hh:mm:ss)> <machine name> <system daemon (sshd)>: Failed password for <user> from <ip address> port <port number> <ssh protocol (ssh2)>
```

#### Failure: invalid / non-existent user

```
<timestamp (m d hh:mm:ss)> <machine name> <system daemon (sshd)>: Failed password for invalid <user> from <ip address> port <port number> <ssh protocol (ssh2)>
```

Attempts by an end user to login with the incorrect port number (but potentially a correct username) will not captured by `rsyslogd`.

Attempts to login to the wrong hostname will also not be captured. 

## Logfile Parsing

`log_parser.py` takes a logfile and timestamp file as input and transmits these results to the database accessible through MongoDB Atlas and the backend API routes ([Dragoffs/SL2/backend/routes](https://github.com/Dragoffs/SL2/tree/main/backend/routes)).

The timestamp file allows the script to determine what should be parsed from the logfile in order to speed up the parsing process.  Correct usage of the API in `log_parser.py` should prevent any duplicate records.

Lines not already parsed are matched against the three login result patterns (success, failure, & invalid user) and login attempts are stored in the following Python dictionary structure:

```
{
	username: [
		{
			datetime: <yyyy-mm-dd>T<hh:mm:ss.uuu>+<HH:MM>,
			result: <"Failure", "Success">
		}
	]
}
```

Where `u` represents one digit in the milliseconds value and `HH:MM` represents a timezone offset of `HH` hours and `MM` minutes.

These objects are "up-serted" (a combination of updating and insertion) into the database, preventing duplicates and inserting new values when appropriate.
