# howdy-solid-design-principles

## Single responsibility principle
Everything is fine so far because the Journal is just handling its primary responsibility, which is the keeping of entries.
```javascript
  class Journal {
    constructor() {
      this.entries = {};
      this.count = 0;
    }

    addEntry(text) {
      let c = ++this.count;
      let entry = `${c}: ${text}`;
      this.entries[c] = entry;
      return c;
    }

    removeEntry(index) {
      delete this.entries[index];
    }

    toString() {
      return Object.values(this.entries).join("\n");
    }
  }

  let j = new Journal();
  j.addEntry("I listened The Old Man's Back Again today");
  j.addEntry(
    "I realized that I love the metaphors Shawn James uses in his songs"
  );
  console.log(j.toString());
```

**But imagine that you decide that you also want to be able to save the journal. Where do you put this functionality?**

It might be really tempting for you to actually add this functionality right into the journal itself. So you might go ahead and let's just import the file system.

```javascript
  const fs = require("fs");

  class Journal {
    constructor() {
      this.entries = {};
      this.count = 0;
    }

    addEntry(text) {
      let c = ++this.count;
      let entry = `${c}: ${text}`;
      this.entries[c] = entry;
      return c;
    }

    removeEntry(index) {
      delete this.entries[index];
    }

    toString() {
      return Object.values(this.entries).join("\n");
    }

    save(filename) {
      fs.writeFileSync(filename, this.toString());
    }

    load(filename) {
      //
    }

    loadFromUrl(url) {
      //
    }
  }

  let j = new Journal();
  j.addEntry("I listened The Old Man's Back Again today");
  j.addEntry(
    "I realized that I love the metaphors Shawn James uses in his songs"
  );
  console.log(j.toString());
```

**But the problem is that subsequently you want to have additional interaction with the file system.**
- You might want to load from a file name, but later on, you might also want to load from somewhere else.

**So the big problem with all of this that you now added a second responsibility to the general class.**
- Why is this a problem?
- Why can't we have a second responsibility?

-----

Well, imagine you have some behaviors which are specific to serialisation, specific to the saving and writing the data.
Let's suppose that instead of doing this.toString(), you want to remove those indices before you have the entry serialized.
And let's suppose you also want to have some processing. When you load that, maybe you save them with the indices, but you load them without the indices.
So you have some common operations.

Now, the problem with all of this is that imagine that a Journal isn't the only object in your system.
Imagine that you have 10 different types that you want to serialize to files and load from files and maybe load from some. You are write somewhere.
**How can you have common operations on all of these? And the answer is, well, it's going to be very difficult for you.**


So it might make sense to take all of these operations related to persistence, because that's what it is.
You might want to take all of them, remove them from this class and just add them to a separate component that can subsequently be generalized for handling different types of objects, not just journal entries, but other things as well.

So a good idea would be to take everything from here and just make a separate class, make a class that's called persistance manager.

```javascript
  const fs = require("fs");

  class Journal {
    constructor() {
      this.entries = {};
      this.count = 0;
    }

    addEntry(text) {
      let c = ++this.count;
      let entry = `${c}: ${text}`;
      this.entries[c] = entry;
      return c;
    }

    removeEntry(index) {
      delete this.entries[index];
    }

    toString() {
      return Object.values(this.entries).join("\n");
    }
  }

  class PersistenceManager {
    preprocess(journal) {
      //
    }

    saveToFile(journal, filename) {
      fs.writeFileSync(filename, journal.toString());
    }
  }

  let j = new Journal();
  j.addEntry("I listened The Old Man's Back Again today");
  j.addEntry(
    "I realized that I love the metaphors Shawn James uses in his songs"
  );
  console.log(j.toString());

  let p = new PersistenceManager();
  const now = new Date();
  let fileName = `journal-${now.getFullYear()}-${(now.getMonth() + 1)
    .toString()
    .padStart(2, "0")}-${now.getDate()}.txt`;
  p.saveToFile(j, fileName);
```
