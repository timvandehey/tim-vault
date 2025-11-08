[public link](https://g.co/gemini/share/f1cb440f4287)

You're looking to parse Markdown files that include YAML front matter (often used in static site generators like Jekyll, Hugo, and 11ty). The front matter is typically delimited by --- at the beginning and end of the YAML block.
Here's how you can do it in Deno, combining file reading with YAML parsing. Since you're asking about Deno, I'll provide the Deno-specific example.
We'll need:
File Reading: Deno's Deno.readTextFile function.
YAML Parsing: Deno's standard library yaml module or npm:js-yaml.
I'll use the Deno standard library @std/yaml as it's the more "Deno native" approach.
Deno Example: Reading Markdown with YAML Front Matter
First, let's create a sample Markdown file with front matter. Save this as example.md:

Markdown


---
title: My Awesome Post
author: John Doe
date: 2025-06-14
tags:
  - deno
  - javascript
  - markdown
published: true
---

# Welcome to my Deno Blog Post

This is the main content of my Markdown file.
It contains some text and perhaps some code blocks.

```javascript
console.log("Hello, Deno!");


This is the end of the post.



Now, here's the Deno JavaScript code to read this file and extract the front matter:

```javascript
// main.ts (or main.js)
import { parse } from "https://deno.land/std@0.224.0/yaml/parse.ts";
// Note: Update the version number to the latest stable version from deno.land/std

/**
 * Reads a Markdown file, extracts YAML front matter, and returns
 * the front matter as a JavaScript object and the remaining markdown content.
 *
 * @param filePath The path to the Markdown file.
 * @returns An object containing `frontMatter` (parsed YAML) and `content` (remaining markdown).
 */
async function readMarkdownWithFrontMatter(filePath: string) {
    try {
        const fileContent = await Deno.readTextFile(filePath);

        const lines = fileContent.split('\n');
        let frontMatterLines: string[] = [];
        let contentLines: string[] = [];
        let inFrontMatter = false;
        let frontMatterDelimitersFound = 0;

        for (const line of lines) {
            if (line.trim() === '---') {
                frontMatterDelimitersFound++;
                if (frontMatterDelimitersFound === 1) {
                    inFrontMatter = true; // Start of front matter
                    continue;
                } else if (frontMatterDelimitersFound === 2) {
                    inFrontMatter = false; // End of front matter
                    continue;
                }
            }

            if (inFrontMatter) {
                frontMatterLines.push(line);
            } else {
                // Only add content after the second delimiter (or if no delimiters were found)
                if (frontMatterDelimitersFound === 2 || frontMatterDelimitersFound === 0) {
                    contentLines.push(line);
                } else {
                    // This handles cases where there's no frontmatter or it's malformed
                    // If we only found one '---' or none, everything is content
                    contentLines.push(line);
                }
            }
        }

        let frontMatter = {};
        if (frontMatterDelimitersFound >= 2) { // Ensure proper front matter block was found
            const yamlString = frontMatterLines.join('\n').trim();
            if (yamlString) { // Only parse if there's actual content in the YAML block
                try {
                    frontMatter = parse(yamlString);
                } catch (yamlError) {
                    console.warn(`Warning: Could not parse YAML front matter in ${filePath}:`, yamlError);
                    // If YAML parsing fails, treat all content as markdown
                    contentLines = lines; // Reset content to include what was attempted as front matter
                    frontMatter = {}; // No front matter successfully extracted
                }
            }
        } else {
            // If no valid front matter delimiters, all content is markdown
            contentLines = lines;
        }


        // Remove any leading empty lines from the content
        while (contentLines.length > 0 && contentLines[0].trim() === '') {
            contentLines.shift();
        }

        return {
            frontMatter: frontMatter,
            content: contentLines.join('\n').trim()
        };

    } catch (error) {
        if (error instanceof Deno.errors.NotFound) {
            console.error(`Error: File not found at ${filePath}`);
        } else {
            console.error(`Error reading file ${filePath}:`, error);
        }
        return { frontMatter: {}, content: '' };
    }
}

// --- Usage ---
const markdownFilePath = './example.md'; // Path to your markdown file

readMarkdownWithFrontMatter(markdownFilePath)
    .then(({ frontMatter, content }) => {
        console.log("--- Extracted Front Matter ---");
        console.log(JSON.stringify(frontMatter, null, 2));

        console.log("\n--- Extracted Markdown Content ---");
        console.log(content);
    })
    .catch(err => {
        console.error("An error occurred:", err);
    });

// To run this Deno script, you need to grant file read permission:
// deno run --allow-read main.ts


Explanation:
import { parse } from "...": Imports the parse function from Deno's standard yaml module.
readMarkdownWithFrontMatter(filePath: string) function:
Deno.readTextFile(filePath): Asynchronously reads the entire content of the specified file as a string.
Splitting by Lines: The file content is split into an array of lines.
Front Matter Delimiter Logic:
It iterates through each line to find the --- delimiters.
inFrontMatter boolean tracks whether the current lines belong to the front matter block.
frontMatterDelimitersFound counts the --- lines.
Lines between the first and second --- are added to frontMatterLines.
Lines after the second --- are added to contentLines.
YAML Parsing: If two --- delimiters are found, the frontMatterLines are joined back into a string, and parse() is used to convert this YAML string into a JavaScript object.
Error Handling: Includes try...catch blocks for file reading errors (e.g., file not found) and YAML parsing errors. If YAML parsing fails, it treats the entire content as Markdown to prevent data loss.
Content Cleanup: Removes any leading empty lines from the extracted content.
Returns an object with the frontMatter (the parsed YAML) and the content (the remaining Markdown).
Running the Script:
You need to save the code as main.ts (or main.js).
Run it from your terminal using deno run --allow-read main.ts. The --allow-read flag is essential because Deno requires explicit permission to access the file system.
This solution provides a robust way to extract YAML front matter from Markdown files in Deno, making it suitable for building tools like static site generators, content management scripts, or any application that consumes Markdown content with structured metadata.
You're combining two useful tasks: reading a Markdown file and extracting its YAML front matter. This is a common pattern for static site generators, content management systems, or any application that uses Markdown files with metadata.
In Deno, the standard library provides a dedicated module for this: front_matter. It works perfectly with the YAML parsing capabilities we discussed.
The typical structure of a Markdown file with front matter looks like this:

Markdown


---
title: My Awesome Blog Post
author: John Doe
date: 2025-06-14
tags:
  - deno
  - javascript
  - markdown
---

# My Awesome Blog Post Title

This is the main content of my blog post. It's written in Markdown.

- Here's a list item.
- Another list item.



Notice the --- (three hyphens) on separate lines that delimit the YAML front matter.
Here's how you can achieve this in Deno:
Using Deno's Standard Library (@std/front_matter and @std/yaml)
Deno's front_matter module is specifically designed to extract this section, and it uses @std/yaml under the hood for parsing the YAML.
1. Create a sample Markdown file (e.g., post.md):

Markdown


---
title: The Deno Journey
author: Dev Wizard
date: 2025-06-14T09:30:00Z
categories:
  - Programming
  - Deno
excerpt: A brief look into building with Deno.
draft: false
---

# Exploring Deno

Deno is a secure JavaScript/TypeScript runtime built with V8, Rust, and Tokio.

It aims to provide a productive and secure scripting environment for modern web development.

## Key Features

- **Security by Default:** Requires explicit permissions for file system, network, etc.
- **TypeScript First:** Native TypeScript support.
- **Single Executable:** Ships as a single binary.
- **Built-in Tools:** Includes a linter, formatter, test runner, and more.


2. Create a Deno script (e.g., read_md_frontmatter.ts):

JavaScript


// read_md_frontmatter.ts
import { extract } from "https://deno.land/std@0.224.0/front_matter/mod.ts";
// The 'mod.ts' exports the main 'extract' function which handles common delimiters (---, +++).
// If you only want YAML front matter, you could also import:
// import { extractYaml } from "https://deno.land/std@0.224.0/front_matter/yaml.ts";

const filePath = "./post.md";

try {
    // Read the content of the Markdown file
    const markdownContent = await Deno.readTextFile(filePath);

    // Extract front matter and body
    // The 'extract' function returns an object with 'attrs' (the parsed YAML) and 'body' (the rest of the Markdown)
    const { attrs, body } = extract(markdownContent);

    // 'attrs' will be a JavaScript object (parsed from YAML)
    const frontMatterJson = JSON.stringify(attrs, null, 2);

    console.log("--- Extracted Front Matter (JSON) ---");
    console.log(frontMatterJson);

    console.log("\n--- Markdown Body ---");
    console.log(body);

} catch (error) {
    if (error instanceof Deno.errors.NotFound) {
        console.error(`Error: File not found at ${filePath}`);
    } else {
        console.error("An error occurred:", error);
    }
}

// To run this script, you'll need file read permission:
// deno run --allow-read read_md_frontmatter.ts


Explanation:
import { extract } from "https://deno.land/std@0.224.0/front_matter/mod.ts";: This line imports the extract function from Deno's standard library's front_matter module.
@std/front_matter specifically looks for delimiters like --- or +++ at the beginning of a string to identify the front matter block.
mod.ts provides a general extract function that auto-detects the format (YAML, TOML, JSON). If you know it's always YAML, you could use extractYaml directly for a slightly more specific import, but extract is usually fine.
Important: Always update the version number (0.224.0 in this example) to the latest stable version of Deno's standard library for security and features. You can find it on deno.land/std.
await Deno.readTextFile(filePath);: This is a Deno-specific API for asynchronously reading the content of a file as a string. Since it's an asynchronous operation, we await its result.
const { attrs, body } = extract(markdownContent);:
The extract function takes the full Markdown string as input.
It returns an object with two properties:
attrs: This is the JavaScript object parsed from the YAML front matter.
body: This is the remaining Markdown content after the front matter block has been removed.
JSON.stringify(attrs, null, 2);: As before, we use JSON.stringify to convert the JavaScript object (attrs) into a human-readable JSON string. The null, 2 arguments ensure it's pretty-printed with 2-space indentation.
--allow-read Permission: When running this script with deno run, you need to explicitly grant it permission to read files from your file system. Deno's security model requires this.
This approach is clean, idiomatic Deno, and leverages well-tested components of the Deno standard library to handle the complexities of parsing both Markdown and YAML.

