{% head %}
{% info %}

{% title-how-to-build-your-own-small-sqlite-clone-sqlcik %}

{% banner-how-to-build-your-own-small-sqlite-clone-sqlcik %}

{% content-start %}

## Introduction
This is my second project after I did a small compiler, and it took more time than I expected. Not because the project was complex or hard, but because I didn't have too much time for it since I was working on my graduation project and finals. Now, as I am a graduated Computer Engineer, maybe it is time to actually take the art of programming and technical stuff more seriously and improve!

## What do you need to build a small database engine?
We can divide the whole project into small parts. I am pretty sure that if you want to build a more complex engine, you would need a better-thought-out structure, but for my case, where I will just support simple operations such as:

- `select` 
- `insert`
- `delete`
- `update`
- `all`

I divided the project into:
```bash
src/
├── database.ts # Where all hard operations take place
├── main.ts # REPL
├── parser.ts # Tokenization of prompts
└── types.ts # Global types
```
and plus `users.json` file for disk storage.

## What is a REPL?
REPL stands for `read, eval, print loop` and it is the heart of our project. `main.ts` contains our REPL and it looks like this:

```ts
import readline from "readline";
import { parseCommand } from "./parser";
import { save, getUser, deleteUser, getAll, updateUser } from "./database";
const rl = readline.createInterface({
    input:process.stdin,
    output:process.stdout,
    prompt:"sqlcik >>> "
});

console.log("Welcome to SQLCIK")
rl.prompt();

rl.on("line",(line) => {
    const input = line.trim();
    if(input === ".quit"){
        rl.close()
        return
    }
    let parsedCommand = parseCommand(input)
    let result = null
    let commandType = parsedCommand.commandType
    if(commandType == "insert"){
        result = save(parsedCommand.user)
    } else if(commandType == "select"){
       result = getUser(parsedCommand.user.id)
    } else if(commandType == "delete"){
        result = deleteUser(parsedCommand.user.id)
    } else if (commandType == "all"){
        result = getAll()
    } else if (commandType == "update"){
        result = updateUser(parsedCommand.user)
    } else {
        result = "Error: undefined command"
    }
    console.log(result)
    rl.prompt()
});

```

## Parsers are everywhere :D

Actually, we are encountering parsers all the time, even in places we sometimes don't expect as normal users. But every time there is something like natural language or structured input, there is always a parser.  

In our case, we need to parse prompts like `insert 0 ata atasoy 22` to an object:
```json
{
  "commandType": "insert",
  "user": {
    "id": 0,
    "name": "ata",
    "surname": "atasoy",
    "age": 22
  }
}
```

A simple function can solve this for us, BUT it gets more complex if prompts are more complicated than simple insert or update operations. 

## Database.ts - where all the hard operations take place

Here we handle all operations regarding the file operations and the logic behind our prompt operations. This file is home to all the main functions used in `main.ts` and is where all the magic happens. The code here is not complex, but I have some design choices I made and want to talk about.

For example, let's take a look at the `deleteUser` function:

```ts
export function deleteUser(id:String): EngineResponse {
    const usersFile = fs.readFileSync('./users.json', 'utf-8');
    const users = JSON.parse(usersFile);
    let usersObjectList = users
    
    for(let i = 0;i<usersObjectList.length;i++){
        if(usersObjectList[i].id === id){
            let deletedUser = usersObjectList[i]
            users.splice(i,1) 
            fs.writeFile("users.json", JSON.stringify(users), 'utf8', (err) => {
            if (err) {
                console.error('Error writing to file', err);
            }
                })
            return {"status":"success","user": deletedUser}
        }
    }
    
    let nullUser:User = {id:"null", name: "null", surname: "null",
        age: "null"}
    return {"status":"failed: provided ID is not valid ", "user": nullUser}

}
```

First of all, you can see that this returns the `EngineResponse` type, where `EngineResponse` is just a status code and a user object.

About the loop where we find the user with the ID and delete them, we could also use a simple `filter` function to very quickly delete the unwanted user, but I didn't think of that at the start. Plus, I would like to return the deleted user before deleting, so this way is good anyway. I don't know which one is more efficient, but this looks like the cleaner option to me.

## Conclusion

To build your own SQLite (simple like mine), you would need to create mainly 3 different files, but the most important things to get from this are:

- What is REPL
- How the parser is working
- And just file manipulations

You should definitely try to build your own SQLite-like project because I think it would really deepen your knowledge about databases, their limits, and the best ways of writing good SQL commands, because you would have more knowledge of how everything is working under the hood.

Thanks for reading :D  
GitHub link: [https://github.com/atasoya/sqlcik](https://github.com/atasoya/sqlcik)

{% content-end %}

{% footer %}
