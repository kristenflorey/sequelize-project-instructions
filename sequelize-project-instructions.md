## Sequelize Project Instructions

##### Goal:
- #### Build a Data Access Layer to power a web application
  - Application: standard express.js application using the pug library to generate the HTML and the node-postgres library to connect to the database.
    - It already has sequelize and sequelize-cli installed.

* Compare controller files in SQL & Sequelize projects

Recipes will contain:
- A title,
- A list of ingredients, and
- A list of instructions.
  - The text of the instruction,
  - The order that it appears in the recipe, and
  - The id of the recipe that it belongs to.
- The date/time that it was entered into the recipe box, and
- The date/time it was last updated in the recipe box.

Structure of each recipe:
- An amount (optional),
- A unit of measure (optional),
- The actual food stuff, and
- The id of the recipe that it belongs to.

<img src="https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Module-SQL/assets/sql-recipe-box-data-model.png" width="500"/>

### Setup
- Clone the starter project from https://github.com/appacademy-starters/sql-orm-recipe-box
- Run `npm install` to install the packages
- Run `npm run dev` to start the server on port 3000
- You'll do all of your work in the **data-access-layer directory**

####1. Initialize the Sequelize project
- type `npx sequelize-cli init`

####2. Using  `psql` or Postbird, create a new user
- for this application named "sequelizerecipe_box_app" with the password "HfKfK79k" _and the ability to create a database
- Change all the "user" and "password" values to the information for the user that you created in Phase 2.
- Change the "database" values to be "recipe_box_development", "recipe_box_test", and "recipe_box_production".
- Change all of the "dialect" values from "mysql" to "postgres".
- Delete all of the "operatorAliases" entries.
  - Make sure to remove the comma from the preceding line so that it's valid JSON.
- add `"seederStorage": "sequelize"` to each of the different blocks

####3. Create your database
- run `npx sequelize-cli db:create`
  - should print:
    ```
    Sequelize CLI [Node: 10.19.0, CLI: 5.5.1, ORM: 5.21.5]

    Loaded configuration file "config/config.json".
    Using environment "development".
    Database recipe_box_development created.
    ```
- run
  ```
  npx sequelize-cli model:generate \
  --name MeasurementUnit \
  --attributes name:string
  ```  
  - should print:
    ```
    New model was created at models/measurementunit.js
    New migration was created at migrations/20200101012349-MeasurementUnit.js
    ```
- Open the new migration file `migrations/20200101012349-MeasurementUnit.js` and make the `name` property 'non-nullable'
  - add the "unique" property set to true to the "name" configuration
  - change the type for "name" from `Sequelize.STRING` to `Sequelize.STRING(20)`.

####4. Run your migration
- run `npx sequelize-cli db:migrate`
  - should print:
    ```
    Loaded configuration file "config/config.json".
    Using environment "development".
    == 20200101012349-create-measurement-unit: migrating =======
    == 20200101012349-create-measurement-unit: migrated (0.021s)
    ```
    -  confirm that the table "MeasurementUnits" is created by running `\dt` in your `psql` command or `SELECT * FROM "MeasurementUnits"`

####5. Create the seed data
- crate a **seeder**  by running `npx sequelize-cli seed:generate --name default-measurement-units`
- insert seed data
  - replace the `up` method with:
    ```js
    return queryInterface.bulkInsert('MeasurementUnits', [
        {
          name: 'cups',
          createdAt: new Date(),
          updatedAt: new Date()
        },
      ]);
    ```
  - two parameters:
    - name of table to insert into
    - array of objects that have property names that match the column names in the table":
    "fluid ounces"
  "gallons"
  "grams"
  "liters"
  "milliliters"
  "ounces"
  "pinch"
  "pints"
  "pounds"
  "quarts"
  "tablespoons"
  "teaspoons"
  ""
  "cans"
  "slices"
  "splash"
- run `db:seed:all`
- run `\dt`

####6 . Recipe table model
- Generate a model for the recipe
- Customize the migration so the "title" column is not nullable
- Run your migration and confirm that you defined it correctly by checking the attributes in the description of the table.

##### Instruction table model
- create table model with specifications:

| Column Name   | Column Type | Constraints  |
|---------------|-------------|--------------|
| id            | SERIAL      | PK           |
| specification | TEXT        | NOT NULL     |
| listOrder     | INTEGER     | NOT NULL     |
| recipeId      | INTEGER     | FK, NOT NULL |

- run migrate generation command:
```
--attributes column1:type1,column2:type2,column3:type3`
```
- modify each of the column descriptors in the migration so that the columns are not nullable
- add a new property to the one for "recipeId" called "references" that is an object that contains a "model" property set to "Recipes"
  - should look like:
  ```js
  recipeId: {
    allowNull: false,
    references: { model: "Recipes" },
    type: Sequelize.INTEGER,
  },
  ```
- run `\dt`
  - You should see all non-null columns and a foreign key between the "Instructions" table and the "Recipes" table.

####7. Create the ingredients model
- create the model and migration with:

| Column Name       | Column Type   | Constraints  |
|-------------------|---------------|--------------|
| id                | SERIAL        | PK           |
| amount            | NUMERIC(5, 2) | NOT NULL     |
| measurementUnitId | INTEGER       | FK, NOT NULL |
| foodStuff         | VARCHAR(500)  | NOT NULL     |
| recipeId          | INTEGER       | FK, NOT NULL |

####8. Seed data for all the tables
- create seeder files
- run `npx sequelize-cli db:seed:all`

####9. Updating models with references
- open the **models/recipe.js** file
  - in the `associate` function, replace the comment with:
  ```js
  Recipe.hasMany(models.Instruction, { foreignKey: 'recipeId' });
  ```
- modify the Ingredient, MeasurementUnit, and Recipe files accordingly with the `hasMany` and `belongsTo` associations, always specifying the name of the foreign key column that binds the two tables together.

####10. Updating models with Validations
- open the **models/instruction.js** file
  - change the code to:
  ```js
  'use strict';
  module.exports = (sequelize, DataTypes) => {
    const Instruction = sequelize.define('Instruction', {
      specification: {gi
        type: DataTypes.TEXT,
        validate: {
          notEmpty: true,
        },
      },
      listOrder: DataTypes.INTEGER,
      recipeId: DataTypes.INTEGER
    }, {});
    Instruction.associate = function(models) {
      Instruction.belongsTo(models.Recipe, { foreignKey: 'recipeId' });
    };
    return Instruction;
  };
  ```
####11. Cascade delete for recipes
-  Open the models/recipe.js file
  - modify the second argument of each of the `hasMany` calls to include two new property/value pairs:
    - `onDelete: 'CASCADE'`
    - `hooks: true`
  - don't delete the `foreignKey` property that you put there in Phase 9.

####12. Building the repositories
- run `npm run dev`
- follow hints to create queries in the following files:
  - recipes-repository.js
  - instructions-repository.js
  - ingredients-repository.js

---
END .
