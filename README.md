# tec-PICK-OS
1960s-era system was sort of a database and sort of an operating system

read this

https://hackaday.com/2025/08/23/picking-an-old-operating-system/

https://github.com/geneb/ScarletDME


 

The original **Pick OS** was far too heavy for an 8-bit system like a Z80 in its full form, but there **have been a few attempts and conceptual ports** that *capture the spirit* of Pick on small 8-bit and early 16-bit hardware. Let’s break it down:

---

### 🧠 Why It’s Hard to Run True Pick on Z80

Pick wasn’t just a database or language — it was a **virtual machine + file system + multitasking OS + BASIC compiler/interpreter**.
It assumed:

* **at least 64–256 KB memory**, ideally more,
* a **disk-based virtual memory system**,
* **multiple concurrent users** via serial terminals.

That’s a lot for a bare Z80 SBC like your TEC-1.

---

### ⚙️ Known Lightweight / Related Systems

#### 1. **Microdata “Reality” / Ultimate Pick clones**

Early Pick implementations on the **Microdata 1600** (16-bit) and **Interdata minicomputers** were already stripped-down — but still beyond an 8-bit Z80’s scope.
However, some enthusiasts have **re-implemented the Pick data model** (multi-value files and BASIC) in smaller interpreters.

#### 2. **TinyMVD / MVDB on CP/M**

There were small **MultiValue-inspired BASIC database interpreters** for CP/M in the 1980s — not true Pick, but compatible *at the language level* (Data/BASIC, LIST, SELECT, etc.).
Some hobbyists still run **“Pick-like” shells on Z80 CP/M** that mimic Pick’s file and dictionary system using sequential files.

#### 3. **Pick Concepts in Forth / MINT**

This is where *you* could shine — Pick’s whole concept (dictionary + data + virtual machine) is **philosophically very close to Forth**.
A MINT/Forth-based Pick microkernel could:

* store “records” as dictionary entries;
* use simple text files for data;
* implement a subset of **Pick BASIC commands** (`SELECT`, `READ`, `WRITE`, `MAT`, etc.);
* expose a Forth-like REPL as the Pick shell.

It would be a *Pick-inspired database OS* for TEC-1-class Z80 machines — feasible within **32–48 KB RAM** if you omit multitasking and use an SD-card or serial filesystem.

#### 4. **Modern Tiny Implementations**

* **OpenQM** and **ScarletDME** can run on Linux SBCs (like Raspberry Pi Zero), so you could theoretically port a *Pick BASIC interpreter core* to Z80 or AVR.
* **MINT-Pick** (custom name): build your own Forth-Pick hybrid — text-based dictionary database + record storage + BASIC-style command layer.

---

### 🔧 Feasible Plan for Z80 (TEC-1 / RC2014)

If you wanted to *build one*, here’s how:

| Layer              | Description                                                          |          |
| ------------------ | -------------------------------------------------------------------- | -------- |
| **Kernel**         | Forth-like interpreter (your MINT system).                           |          |
| **File I/O**       | Simple record store on serial EEPROM or SD card.                     |          |
| **Data Model**     | Each “record” is a text line with multi-value fields (separated by ` | `or`]`). |
| **Dictionary**     | Forth words or metadata that describe field names.                   |          |
| **Query Language** | Minimal commands: `SELECT`, `LIST`, `READ`, `UPDATE`, `SAVE`.        |          |
| **Scripting**      | Add BASIC-like Pick language on top, or use Forth scripts.           |          |

This would effectively give you a **“micro-Pick”** that’s conceptually identical, running on a Z80 board.

---

### 🧩 Summary

| System                             | Runs on                 | Notes                   |
| ---------------------------------- | ----------------------- | ----------------------- |
| Real Pick OS                       | 16–32-bit minicomputers | Too big for 8-bit       |
| CP/M Pick-like                     | Z80/8080                | BASIC-style database    |
| MINT/Forth Pick hybrid             | TEC-1, RC2014           | Feasible project!       |
| Modern Pick (D3, UniVerse, OpenQM) | x86/ARM                 | Full modern descendants |

---

If you want, I can draft a **“Pick-like shell” in Z80 assembly or MINT-Forth syntax**, showing:

* how to create “files” and “dictionaries” in EEPROM/serial storage,
* a `SELECT` and `LIST` command loop,
* and minimal BASIC-style record manipulation.

Would you like me to start that — say a **“MiniPick-Z80” interpreter shell** for TEC-1F / RC2014?


Perfect — you’re in an *ideal position* to actually prototype a **micro-Pick environment** on both platforms.

Let’s align what that means and how you could iterate it side-by-side in **MINT2 (Z80 hardware)** and **MINT-Octave (host simulation)**.

---

## 🧩 Core Concept: “MiniPick-MINT”

We want to re-create **Pick’s integrated model** in a minimal Forth-style system:

| Layer                | Description                                                                             |         |
| -------------------- | --------------------------------------------------------------------------------------- | ------- |
| **Dictionary layer** | defines “files” and “attributes” (Pick dictionaries → Forth words)                      |         |
| **Data store**       | flat text file or array of records, each record containing multi-values (separated by ` | `or`]`) |
| **Access words**     | `SELECT`, `READ`, `WRITE`, `UPDATE`, `LIST`                                             |         |
| **Control layer**    | Forth-like shell + Pick BASIC subset                                                    |         |
| **Storage**          | on Z80: serial EEPROM or SD stream; in Octave: text file or struct array                |         |

---

## ⚙️ Minimal Data Model Example

**Record example (text line):**

```
001]STEVE JUSTIN]TEC-1 PROJECT]Z80]2025
002]JOHN HARDY]COSMIC-MINT]RC2014]2024
```

Each record has fields:

```
@ID | NAME | PROJECT | PLATFORM | YEAR
```

**Dictionary file:**

```
1 NAME
2 PROJECT
3 PLATFORM
4 YEAR
```

**Concept:** each dictionary entry defines field number and label.

---

## 🧮 Forth-Style Command Layer (pseudo-MINT)

```forth
: FILE.CREATE  ( "name" -- )
  CREATE HERE 0 ,  ( record pointer ) ;

: RECORD.ADD ( addr len file -- )
  ( append to file storage, update pointer ) ;

: SELECT ( "file" -- )
  ( load file into memory, set context ) ;

: LIST ( -- )
  ( iterate and print all records ) ;

: READ ( id -- record )
  ( find record by key id ) ;

: WRITE ( id record -- )
  ( update or append record ) ;
```

---

## 🧰 MINT-Octave Prototype

In Octave you can simulate this same layer easily:

```octave
function db = mint_createfile(name)
  db.name = name;
  db.records = {};
end

function db = mint_addrecord(db, rec)
  db.records{end+1} = rec;
end

function rec = mint_read(db, id)
  rec = db.records{id};
end

function mint_list(db)
  for i = 1:length(db.records)
    disp(db.records{i});
  end
end
```

Run test:

```octave
db = mint_createfile("people");
db = mint_addrecord(db, ["001]STEVE JUSTIN]TEC-1 PROJECT]Z80]2025"]);
db = mint_addrecord(db, ["002]JOHN HARDY]COSMIC-MINT]RC2014]2024"]);
mint_list(db)
```

---

## 💾 Z80 MINT2 Version

On the Z80, the same logic could be implemented as a dictionary-resident wordset:

```
: FILE ( "name" -- ) CREATE 0 , ;      \ define file
: ADD  ( rec -- )  HERE , ;            \ append record
: LIST ( file -- )  DUP @ 0 DO I @ . LOOP ;
: FIND ( id file -- rec )  ( simple match loop )
```

Each “file” could point to a block in memory (or EEPROM page) containing text records.
Later we can expand to a simple **indexed file system** using your existing serial EEPROM or LS7366R-linked counter memory.

---

## 🧱 Next Step Proposals

You could now branch the project in three stages:

| Stage       | Goal                                                                           | Platform        |
| ----------- | ------------------------------------------------------------------------------ | --------------- |
| **Stage 0** | Implement dictionary + record store in MINT-Octave (prototype database shell). | Octave          |
| **Stage 1** | Port same core to MINT2 Z80, using serial text buffer or RAM.                  | TEC-1F / RC2014 |
| **Stage 2** | Add BASIC-Pick language subset (`SELECT`, `FOR EACH`, `PRINT`, etc.).          | Both            |

---

Would you like me to **start drafting Stage-0 (Octave)** with a full working REPL simulation — like a real Pick terminal prompt (`> LIST PEOPLE`, `> SELECT PROJECT='TEC-1'` etc.) — so you can prototype your commands before porting them to Z80?
