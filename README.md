# froked @zsajidleo/mongoose-state-machine from @safer-bwd/mongoose-state-machine

[![Build Status](https://travis-ci.com/zsajidleo/mongoose-state-machine.svg?branch=master)](https://travis-ci.com/zsajidleo/mongoose-state-machine)

A Mongoose plugin that implement a _state machine_ into a Mongoose schema.

The plugin is based on [javascript-state-machine](https://github.com/jakesgordon/javascript-state-machine).

## Install

```sh
npm install @zsajidleo/mongoose-state-machine --save
```

## Options

- `stateMachine` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** The state machine declaration object ([javascript-state-machine](https://github.com/jakesgordon/javascript-state-machine))
- `fieldName` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** The name of the schema field that stores the current state (optional, default `status`)

## Usage

First you need to declare a schema and extend it using the plugin.

**Important:**
The following schema paths cannot be used: _state_, _is_, _can_, _cannot_, _transitions_, _allTransitions_, _allStates_ and transitions names.
Because they conflict with [javascript-state-machine](https://github.com/jakesgordon/javascript-state-machine)

```javascript
import mongoose from "mongoose";
import stateMachinePlugin from "@safer-bwd/mongoose-state-machine";

// schema declaration
const matterSchema = new mongoose.Schema({
  matterState: String, // field for storing state
});

// state machine declaration
const stateMachine = {
  init: "solid",
  transitions: [
    { name: "melt", from: "solid", to: "liquid" },
    { name: "freeze", from: "liquid", to: "solid" },
    { name: "vaporize", from: "liquid", to: "gas" },
    { name: "condense", from: "gas", to: "liquid" },
  ],
  methods: {
    onMelt() {
      console.log("I melted");
    },
    onFreeze() {
      console.log("I froze");
    },
    onVaporize() {
      console.log("I vaporized");
    },
    onCondense() {
      console.log("I condensed");
    },
  },
};

// extend schema with the plugin
matterSchema.plugin(stateMachinePlugin, {
  fieldName: "matterState",
  stateMachine,
});

const Matter = mongoose.model("Matter", schema);
```

Now you can use [javascript-state-machine](https://github.com/jakesgordon/javascript-state-machine) API after created or retrieved a document from a database.

**Important:**
The plugin does not manipulate data in a database. To save state in the database you need to use Mongoose API.

```javascript
// create document
const matter = new Matter();
matter.matterState; // solid (init state);
matter.is("solid"); // true
matter.is("liquid"); // false
matter.can("melt"); // true
matter.can("freeze"); // false

// transition
matter.melt(); // I melted!
matter.matterState; // liquid
await matter.save(); // save state

// retrieving from a database
const found = Matter.findById(matter.id);
found.matterState; // liquid;
found.vaporize(); // I vaporized!
found.matterState; // gas
found.melt(); // throw error

matter.matterState = "solid"; // gas (no effect!)
```
