# Obsidian_QuickAdd_JavaScript

# Problem

I kept creating dangling orphan sad-faced notes, and losing those all the time..

# Idea

## In general

I want to instantiate a new note, and I want Obsidian to prompt me to link it to another note.

## More specifically

__I want to create a new note, and I want to put links to other notes in the properties of the new note.__ I first learned about this in [No Boilerplate's Obsidian video](https://youtu.be/B0yAy2j-9V0?si=flXzJy_oplntQyHZ&t=450) [^1]. 
[^1]: (The shown syntax is a little off: `[[` and `"` should be in reverse order, like this: `"[[linked_note_name]]"`).

I want to do 3 things:
1. I want to create a new note.
2. I want to be prompted to put a link to a 'vertical / higher' level note (called a 'MOC', for ['Map of Content'](https://youtu.be/gXvozu3I4K0?si=WJJTJmXIzFKDWIwk)), in the properties of the new note.
3. I want to be prompted for a 'horizontal' level note (called 'related'), and have that also be in the properties.

# Solution

[QuickAdd](https://github.com/chhoumann/quickadd) will be used to instantiate a simple template from the template folder. It will then show me a list of all my current notes, of which I can select one for the MOC-linked-note, and then again for the related-linked-note.

## The Template

The template looks like this:

![QuickAdd_12_Note_Template_to_be_instantiated](https://gist.github.com/user-attachments/assets/2ed52f1b-e03b-4977-916a-dfdcabbeddb2)

### Important

1. The words (exact, inc capitalisation) of the properties 'MOC: ' and 'related: '. If you use other words, amend them as well in the corresponding 'Capture' sections later.
2. _One blank line under each of these properties!_

(When using [Linter](https://github.com/platers/obsidian-linter), make sure to disable automatic linting of this file / or all the files in the Templates folder).

## Script to get all the existing notes

Stored somewhere in the Obsidian vault. Put it in a 'Scripts' folder for all I care. Save it with a useful name, ending in `.js`.

Mine is called `selectSourceFile.js`.

``` javascript
module.exports = async function(params) {
  // Debug notification to confirm script is running
  // new Notice("Script started");
  
  try {
    // Get all markdown files
    const allFiles = params.app.vault.getMarkdownFiles();

    // Debug
    // new Notice(`Found ${allFiles.length} files`);
    
    // Show the suggester
    const selectedFile = await params.quickAddApi.suggester(
      (file) => file.basename,
      allFiles
    );
    
    if (!selectedFile) {
      new Notice("No file selected");
      return;
    }
    
    // This is where you create the variable 'selectedNote' (it could've been called anything).
    // It is also setting it as the output of this script (to be used later in QuickAdd) by assigning it to the params.variables
    // In QuickAdd, you use this output as it's input by using `{{value:selectedNote}}` in the block after where this file runs. Note the lack of the full path (params.variables…) when using in QuickAdd!
    params.variables = { selectedNote: selectedFile.basename }; 

    // This is how you use that same variable in this same script. Note the full 'path' (params… etc).
    // new Notice(`Selected: ${params.variables.selectedNote}`);

  } catch (error) {
    new Notice(`Error: ${error.message}`);
    console.error(error);
  }
}
```

This script will get all the notes in your vault, and display them as a dropdown list from which you can type / search / choose.

Keep an eye out for the `params.variables` and then specifically the `selectedNote` part. That will be important later.

## Setting it up

Below is the 'Macro Manager' window in QuickAdd.

![QuickAdd_11_Macro_Manager](https://gist.github.com/user-attachments/assets/b18a656e-42f3-430c-b938-761e9bd70b9a)

It is the <u>macro</u> 'New_QuickAdd' that I want to use. QuickAdd has a seemingly 'double' feature where you create a macro, and expose that separately. Very confusing.

The macro is exposed below in the QuickAdd Settings. You do this by typing a name (in my case I used the same name), selecting Macro, and hitting Add Choice. Then go to the settings of the thing you just created (gear icon) and select the _macro_ called 'New_QuickAdd' to this exposed variable 'New_QuickAdd'.

Exposing it here allows two things:
1. The macro can be used by other macros / templates / captures in QuickAdd
2. The rest of Obsidian can use it.
	1. It is always possible with Command Palette (Run QuickAdd > New_QuickAdd)
	2. The yellow lightning bolt also makes it possible to assign a hotkey.
	
![QuickAdd_1_Settings](https://gist.github.com/user-attachments/assets/d1fefd9e-5a51-4ada-882a-0caaa2aec881)

The exposed New_QuickAdd is using the macro New_QuickAdd.

![QuickAdd_2_NewQuickAdd](https://gist.github.com/user-attachments/assets/43bfce1a-e6ce-494b-9396-127143204c03)

## The <u>macro</u> New_QuickAdd

When you hit the gear icon in the exposed thing (or via the Macro Manager), you can change the settings of the macro.

The macro New_QuickAdd uses a ('local') Template option to create a new file (hit the 'Template' button right there). I say 'local' because it is also perfectly happy to use an already created QuickAdd template or Capture. However, then you'd end up with even more separate things in these menus here.

Our macro also uses two other macros (Select_MOC and Select_Related). The latter two create the linked note in the properties of our new note. The order is important: create file first, add properties later.

![QuickAdd_3_NewQuickAdd_details](https://gist.github.com/user-attachments/assets/38e03912-fb8f-4da1-93fe-d67debd516a7)

The New_File creates a file using the template selected from the Templates folder. The template I want to use is the one seen earlier. The other thing to see here is the `{{value: New File Name}}`. This prompts you to type in a textbox, with a little header above saying 'New File Name'. _Important is the space!_ `{{value: New File Name}}` and not `{{value:New File Name}}`.

![QuickAdd_4_NewFile](https://gist.github.com/user-attachments/assets/669ef43f-75ee-491d-8edd-7bf75f758b49)

(A little foreshadowing, later we do something similar when getting input via the `.js`. _However_ then the space is __not__ allowed, and it is this one fucking space that took me 5 hours to realise why I was messing this up).

## The exposed Select_MOC

Below you see the other macro being exposed.

![QuickAdd_5_SelectMOC](https://gist.github.com/user-attachments/assets/8e466dd2-195f-4bab-9c8d-45d55c89b9e4)

## The Select_MOC macro

Very important is the order! The first thing it uses is the 'User Script' `selectSourceFile.js` we saw before. The second thing is a 'local' Capture where it uses the script's output as it's own input. You get this 'local' Capture by hitting the big Capture button right there.

![QuickAdd_6_SelectMOC_details](https://gist.github.com/user-attachments/assets/b49cd975-2a43-4d56-a523-cd6bcd660ff6)

## The Capture

This gets a selected file from the `.js` script, and puts it as a link into the properties / frontmatter of the newly created note.

Capture to active file (the newly created file):

![QuickAdd_7_GetMOC_from_script](https://gist.github.com/user-attachments/assets/ef1a7a26-9a5e-4131-ae88-51bb65541eb4)

__Important:__ The thing you type below 'Insert after' must have the exact wording / capitalisation of one of the properties found in the note template (in this case 'MOC:').

Because of the 'Insert after' is also why we needed to have that __blank line__ underneath this property. If you don't, then the properties section get's messed up.

### Getting the selected note into the Capture

The way to get the output from the `selectSourceFile.js` to our Capture, is to use `{{value:selectedNote}}` in the Capture Format. _(See that spacing right there? 5 hour spacing, that is!)._

The `selectedNote` part comes from what the script outputs via it's `params.variables`. So because the script sets the `params.variables.selectedNote` as the name of the note that we selected, we import that value here this way. If the script had set the note name to `params.variables.buttsoup`, you would have used `{{value:buttsoup}}` here instead.

__!!IMPORTANT SPACING IN THE CAPTURE FORMAT!!:__ there is __no__ space between `value` and `selectedNote`. So use `{{value:selectedNote}}` and not `{{value: selectedNote}}`. Five hours, ladies and gentlemen.

The `[[` and `]]` brackets surrounds the note name to make it a link to our selected note, pretty standard Obsidian stuff.

Now the `"` and `"` surround that yet again because… I don't quite know actually? It has to do with the note being used as a link inside of the _properties_ of a new note. If you'd simply used the link in the body of the note, the `"` wouldn't have been needed.

In summary: Your Capture input should match the `.js`'s output. So for this situation (inc. making it a link in the properties) it should be `"[[{{value:selectedNote}}]]"`, but in another scenario it might be [`[[{{value:pickedFilePath}}]]`](https://forum.obsidian.md/t/quickadd-macro-showing-a-list-of-files-to-capture-automatically/41185/10), or `{{value:buttsoup}}`.

## Now we do the same thing but now for 'related' note

Now we're getting the 'related' note as a link into the frontmatter.

This is the exposed macro:

![QuickAdd_8_SelectRelated](https://gist.github.com/user-attachments/assets/7cbbf4c4-6697-49f8-b51a-6f62ddf62b4c)

Below is the macro itself. See that we're using the `selectSourceFile` script here yet again.

![QuickAdd_9_SelectRelated_details](https://gist.github.com/user-attachments/assets/1fd6aa5b-50ea-4147-b45c-f7b808ca5658)

The image below is the 'local' Capture. Again we capture to active file. We're using the property name ('related:') from the template here exactly. And again, theres __no space between `value` and `selectedNote`.__

![QuickAdd_10_GetRelated_from_script](https://gist.github.com/user-attachments/assets/7a5e2c15-815c-4f41-9108-bcbc3040a968)

# Final words and possible improvement

This should work!

You can even duck out of adding either field (with escape). However, in future versions I'd want to make it a little nicer, because it throws some error messages when you do.

The option to add multiple 'related' notes could be nice as well.
