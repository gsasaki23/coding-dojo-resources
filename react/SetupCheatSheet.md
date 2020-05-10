# Setup Cheat Sheet for MongoDB + Express + React
**_NOTE: Double check and change anything with a {{ }}!_**
- [Original Sheet from Coding Dojo](https://github.com/TheCodingDojo/student_md_docs/blob/master/CA-OC/MERN/cheat_sheet.md)

## 1. Backend / Server
1. Make project folder in desired location, with `server` folder nested inside
2. `cd server`
3. To make new back-end node project with `package.json`, run command: `npm init -y`
3. Install dependencies: `npm install --save express mongoose nodemon cors`
4. Manually create `server.js` file
5. Create modularized back-end structure with `config`,`models`,`controllers`,`routes` folders and files.

* To start, check that terminal is inside `server` level, and run command: `nodemon server.js`
* Test CRUD and validations using Postman before moving on to client

---

## 2. Frontend / Client
1. `cd` back to project level
2. To make new front-end react project named `client`, run command: `npx create-react-app client`
3. Go refill coffee
4. Open a 2nd terminal, run `cd client`
5. npm install --save axios @reach/router

* To start, run command: `npm start`

---

## Troubleshooting
* Make sure that mongo itself is running. Check in `Services`
  - To manually start it, `cd C:\Program Files\MongoDB\Server\4.2\bin`, `mongo.exe`
  - Ready to go when cursor becomes `>_`