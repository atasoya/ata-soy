{% head %}
{% info %}

{% title-compiler-construction-md2jsx %}

{% banner-compiler-construction-md2jsx %}

{% content-start %}

## Introduction
I had enough of vibe coding for around two months, so I wanted to take control back and learn more about computer science. So I built a simple Markdown to JSX compiler. I'm currently studying programming language fundamentals where we also go through this stuff! So it was the perfect time to get my hands dirty with it.

So the challenge is:
- No AI (not even for general stuff)
- I can only use NVIM (took 4 hours to set up)
- I have to understand everything and write a blog post!

Let's get started. 

## What is a compiler?

Software that translates files from one language to another (Markdown to JSX in our case). A typical example would be the Go compiler, where Go source code is compiled into executable machine code.

It has several phases and components. Here are the main ones:

**Lexer** (scanner):
- Scans the file and generates tokens from the input
- Example: `let x = 5;` → `["let", "x", "=", "5", ";"]`

**Parser** (analyzer):
- This is where the AST (Abstract Syntax Tree) is built using grammar rules we covered in automata classes :D
- In my case, I don't need a detailed analyzer since my .md to .jsx implementation is fairly small.

**Code Generation**:
- Generates the actual code from the parsed AST
- The easiest part :D

## md2jsx Outline
So here is the outline of the project:

I have 3 main files:
- `lexer.ts`: where I scan and generate the AST (has two parts: block and inline)
- `generator.ts`: creates JSX from the AST
- `main.ts`: glues everything together

## Lexer
First of all, we should define our types:

```ts
type token =
  | { type: "header"; level: number; content: string }
  | { type: "paragraph"; content: inline_token[] };

// inline tokens
type inline_token =
  | { type: "bold"; content: string }
  | { type: "highlight"; content: string }
  | { type: "normal"; content: string };

// modifier dict
const modifiers: { [key: string]: string } = {
  "*": "bold",
  "=": "highlight",
};
```

For this small compiler, I will add support for headers and some inline modifiers like `*` and `=`.

### Diving into the lexer function

As the name _scanner_ suggests, first we should scan the file and start iterating through it. I decided to use the Deno runtime for this project!

```ts
const content = await Deno.readTextFile("example3.md");
```

I need some way to determine whether we're at a header or a paragraph. My idea was to check if the character is `#` - if so, it means we're at a header. But this implementation is prone to bugs and edge cases. For example, if `#` appears in a paragraph, our compiler could break :((

```ts
// get header level
function get_header_level(index: number, content: string): number {
  if (peek(index, content) != "#") {
    return 1;
  }
  return get_header_level(index + 1, content) + 1;
}
```

Here is my cool recursive function to get the header level (`#` → 1, `##` → 2, `###` → 3). After getting the level of the header, we should _eat_ (skip) those indexes so we don't repeat ourselves. For example:

```ts
const header_level = get_header_level(i, content); // calculate header level eg. # ## ###
i += header_level; // jump so we don't check # again
```

If we're not at a header, it means we're scanning a paragraph. Here, we should also handle modifiers like **bold** and ==highlight==.

```ts
if (modifiers[char] != undefined) { 
// checking if char is modifier
        // check if normal content exists
        if (paragraph_content != "") {
          let sub_inline_token: inline_token = {
            type: "normal",
            content: paragraph_content,
          };
          inline_tokens.push(sub_inline_token);
          paragraph_content = "";
        }
        let sub_content_data = get_sub_content(i, content, char);
        let sub_inline_token: inline_token = {
          type: modifiers[char],
          content: sub_content_data[0],
        };
        inline_tokens.push(sub_inline_token);
        i += sub_content_data[1];

        continue;
      }

      // normal text
      paragraph_content += char;
    }
```

## Generator

![AST output screenshot](../public/compiler-construction-md2jsx/ast-screenshot.png)

This is our AST output from the lexer. Now we need to generate JSX from it. The generator is pretty straightforward. I chose to use `map` methods to handle this, but there are other different and possibly easier ways as well.

```ts
export function generator(ast: Token[]): string {
  const jsx = ast.map((node) => {
    switch (node.type) {
      case "header":
        return `<h${node.level}>${node.content}</h${node.level}>`;

      case "paragraph":
        const inner = node.content
          .map((sub_node) => {
            switch (sub_node.type) {
              case "normal":
                return sub_node.content;
              case "bold":
                return `<strong>${sub_node.content}</strong>`;
              case "highlight":
                return `<mark>${sub_node.content}</mark>`;
            }
          })
          .join("");
        return `<p>${inner}</p>`;
    }
  });

  return `export default function Md2jsx(){
      return (${jsx.join("\n")} )}`;
}
```

## Summary
So, to build a compiler, you need to code a lexer, parser, and generator. If you are building a fairly simple compiler, you might not need a separate parser for syntax analysis and AST generation.

In my project, I'm scanning (lexing) a `.md` file and generating `.jsx` from the AST it produces. It's kind of extensible with token types, and it can even support custom tokens and styles.

My main motivation for this is that I write my blogs in Markdown, but I want to host them as static files in my Next.js app. So this is a great starting point for that!

You can check out the whole project on my GitHub: [https://github.com/atasoya/md2jsx-compiler](https://github.com/atasoya/md2jsx-compiler)

{% content-end %}

{% footer %}
