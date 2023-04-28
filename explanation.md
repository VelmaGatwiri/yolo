- Used NPM ci to install dependencies and used CMD NPM to start the application
- Created a docker-compose.yml file where I connected the database to the backend.
- The backend depends on the mongoDB and the frontend is dependent on the backend
- Client is running on port 3000 and the backend on 5000.
- I used volumes to ensure that the database persists.
- Tagged the images to display v0.1

# ERRORS
- Some of the errors experienced was the client side not staying active after creation. To mitigate that I set tty to TRUE.
- Another error was on the port allocation. I had set backend to listen on 8080 which was used by Jenkins so it was not working.

# SIZE REDUCTION
- Created the client and backend dockerfiles based on the node alpine image. I used node because the languages used ae react and node and I used Alpine to reduce image size.
- Used .dockerignore file to prevent running of unnecessary files.
- Reduced the number of layers required to build the image to the least possible amount.
- Used NPM ci instead of npm install and put the environemt as production.
