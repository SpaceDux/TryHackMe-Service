# Hello TryHackMe Team.

_Task completed by Ryan Williams_.
_Task Date: 06/12/2023_.

# Summary

TryHackMe required me to build a full stack todo list application. This was the briefing;

> Your task is to create a simple full-stack web application for managing tasks. The
> application should allow users to create, read, update, and delete tasks. You will need to
> use Node.js and Express for the backend, React for the frontend, and MongoDB for the
> database.

## Setup instructions

Clone this repository, as it will contain everything you need to make it work as its using GitHub submodules, which link to the backend & frontend repositories.

Once you have cloned this repository, you'll need to ensure you have docker installed and running.
In your terminal cd into the repository folder and run the following commands in the order they are displayed.

`git submodule update --init --recursive`
`git submodule foreach "git switch master && git pull && npm i`
`docker compose up` - This is not in detached mode, so cancelling in your terminal will down the containers. add `-d` to detach.

Once docker has built your containers, you'll then be able to navigate to the running app. which should be located at `http://localhost:3000` - Backend
Open a new terminal, run `cd TryHackMe-Frontend && npm run dev`, this will boot up your frontend end which is located at `http://localhost:5173`.

MongoDB will also be running on its usual port, there is no authentication required.

If you would like to browse the individual repositories;
https://github.com/SpaceDux/TryHackMe-Frontend
https://github.com/SpaceDux/TryHackMe-BackEnd

### Backend Requirements:

1.  Create a new task.
2.  Edit a task.
3.  Retrieve a list of tasks.
4.  Delete a task.

### Frontend Requirements:

1. Creating a new task.
2. User friendly way to display existing tasks.
3. The ability to edit or delete tasks.

### Technical Requirements:

1. Use NodeJS and Express to create the backend API endpoints.
2. Use React to create the frontend.
3. Use MongoDB to store the tasks.
4. Use TypeScript.
5. Use Git for version control and commit your changes to a public repository on GitHub.

# How I completed this task.

Firstly I evaluated the tech spec, and established that I will be building this in 3 layers, and we need all 3 layers to work in tandem, so I created GitHub repositories for the frontend & backend. I then created a third repository which I will use as a monorepo and add in my two front/back-ends as submodules, with this I then created a simple docker-compose file to handle both applications as containers, as well as building a MongoDB container for our database.

### The backend;

As we will be building this in TypeScript, I decided to leverage the power of "Decorators" to make my code more readable for the reviewing party. The decorators I added where essentially wrappers on Express route methods.

Example;
`@Controller('<INSERT BASE ROUTE URI HERE>')`
`@<GET | POST | PUT | DELETE>('<INSERT URI HERE')`

As well as the decorators, I also installed Prisma as my database ORM, there was no particular reason I wen't with Prisma over any other tool such as; mongoose, it was merely down to preference.
My file structure is also chosen out of preference, it's just the way I've been writing my code for a long time.

I build 1 controller class which will handle all our routes. The routes are;
`GET /api/task/list` - Fetch a list of tasks.
`GET /api/task/:id` - Fetch an individual task by id.
`PUT /api/task/update/:id` - Update a task.
`POST /api/task/create` - Create a new task.
`DELETE /api/task/:id` - Delete a task.

Within the repository methods for create/updating a task, I leverage the power of transactions to ensure that if an error occurs somewhere in the chain, we will exit out of the transaction and no data will be committed to the database. I understand this might be overkill for such a basic application, but there are a few moving parts in these functions, so I decided it would make sense to wrap them up in cottonwool.

#### What could I do better?

Better validation, I built this super quickly, so I glossed over the data validation, in a real-world application, this would absolutely not be tolerated, but I already found myself spending a lot more time than necessary building up the boilerplate code to make my experience nicer, as-well as the reviewers.

---

### The Frontend;

I setup a basic TS+React+Vite app using `npm create vite`. Once the base app was setup, I began stripping pieces out, removing the setup files, and building my own structure.
I tend to isolate my components in a /src/features directory, it allows my to easily maintain my code as its split into smaller modules, so with this I created a task feature.

Once I was happy with my basic file structure, I began to plan how I was going to achieve this task.

I decided for ease, to use Chakra-UI library, this library gives us lots of components as-well as easy theming if required, there are other UI libraries out there, but Chakra is my usual go-to.
I then needed some form of validation tooling, I've used a few different validation/schema building tools before, Yup, Zod etc. I decided to go with Zod, a TypeScript-first schema validation with static type inference and has zero dependancies (bonus!).
I also wanted to do this slightly different as to your usual TODO list applications, I wanted to build a simple yet effective kanban board to manage the state of my todo cards. So to achieve this I sourced a wonderful OS tool built by the folks at Atlassian, this tool gives us an easy to use drag+drop components that will essentially turn out very simple todo list columns into drag/droppable kanban lanes in very little code. (woo!)

I also decided that I would use a small state management library called 'react-sweet-state', this will handle the reactivity of changes that happen to the tasks state.

I began creating the simple UI components, such as a card to display the todo record, the modals to update, delete and create new todo's.
Located in `src/features/task/components`.

I then built a useTask hook where I will handle all the API calls & validation checks using Zod schemas. I added a react state value called "loading", this will be used within all the components to provider the user feedback that/when something is happening.

I build 4 functions;

1.  getTasks - calls `/api/task/list`
2.  updateTask - calls `/api/task/update/:id`
3.  createTask - calls `/api/task/create`
4.  deleteTask - calls `/api/task/:id`
    Located in `src/features/task/hooks`

I then built a simple page to bring all these components into action, aswell as building the three lanes I will be using (Todo, Active, Complete).

After scaffolding up a simple view, I then began implementing the earlier built feature components, once I was happy with the UI, I then began implementing the drag & drop functionality which was surprisingly very straightforward, the package was well documented, with some examples also available.

We now have a working Kanban board, where I can simply drag a card to a new lane, which will update the tasks status depending on its lane. I can also click on a card which will open a modal to update data, and also delete a todo.

### What could I do better?

I would like to expand the features of this app, adding comments, persisting the order of the cards in the lanes. I actually quite enjoyed working on this piece, so I might still actually implement this if I have the time.
