<template>
  <div class="container">
    <div class="radio-group">
      <label>
        <input type="radio" v-model="selectedDataset" value="all"> All Monster
      </label>
      <label>
        <input type="radio" v-model="selectedDataset" value="world"> Monster Hunter World Monster
      </label>
    </div>
    <button class="solve-button" @click="revealMonster">Reveal Monster</button>
    <h1>Monsterdle</h1>
    <h2 v-if="monsterRevealed">{{ randomMonster.name }}</h2>
    <label for="inputField">Enter monster name:</label>
    <input v-model="inputValue" type="text" id="inputField" @input="filterSuggestions">
    <ul v-if="showSuggestions" class="suggestions">
      <li v-for="(suggestion, index) in filteredSuggestions" :key="index" @click="selectSuggestion(suggestion.name)">{{ suggestion.name }}</li>
    </ul>
    <button @click="addRow" :disabled="!isValidInput">Add to Table</button>
    <button class="new-guess-button" @click="newGuess">New Guess</button>

    <table>
      <thead>
        <tr>
          <th class="name-cell">Monster</th>
          <th>Type</th>
          <th>Elements</th>
          <th>Weaknesses</th>
          <th>Size</th>
          <th>IconColor</th>
          <th>First Appearance</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in rows" :key="index">
          <td :style="getBackgroundStyle(row.name)" class="monster-cell">{{ row.name }}</td>
          <td :class="['table-cell', getCellClass(row.type, randomMonster.type)]">{{ row.type }}</td>
          <td :class="['table-cell', getCellClass(row.elements, randomMonster.elements)]">{{ formatTypes(row.elements) }}</td>
          <td :class="['table-cell', getCellClass(row.weaknesses, randomMonster.weaknesses)]">{{ formatTypes(row.weaknesses) }}</td>
          <td :class="['table-cell', 'size-cell', getCellClass(row.size, randomMonster.size)]">
            <span class="size-text">{{ row.size }}</span>
            <span v-if="row.size > randomMonster.size" class="arrow down"></span>
            <span v-else-if="row.size < randomMonster.size" class="arrow up"></span>
          </td>
          <td :class="['table-cell', getCellClass(row.iconColor, randomMonster.iconColor)]">{{ formatTypes(row.iconColor) }}</td>
          <td :class="['table-cell', getCellClass(row.firstAppearance, randomMonster.firstAppearance)]">{{ row.firstAppearance }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
const monstersData = require('./monster.json');
const monstersWorldData = require('./monsterWorld.json');
const unknownImage = require('../icons/unknown.png');

export default {
  data() {
    return {
      inputValue: '',
      rows: [],
      guessedMonsters: [],
      suggestions: monstersData,
      selectedDataset: 'all', // Default to 'All Monster' dataset
      filteredSuggestions: [],
      showSuggestions: false,
      randomMonster: {},
      monsterRevealed: false,
      cellHeight: '60px' // Adjust as needed for uniform cell height
    };
  },
  computed: {
    isValidInput() {
      return this.filteredSuggestions.some(suggestion => suggestion.name.toLowerCase() === this.inputValue.toLowerCase() && !this.guessedMonsters.includes(suggestion.name.toLowerCase()));
    }
  },
  created() {
    this.randomMonster = this.getRandomMonster();
  },
  watch: {
    selectedDataset() {
      this.updateDataset();
      this.newGuess(); // Trigger selection of new random monster when dataset changes
    }
  },
  methods: {
    updateDataset() {
      if (this.selectedDataset === 'all') {
        this.suggestions = monstersData;
      } else if (this.selectedDataset === 'world') {
        this.suggestions = monstersWorldData;
      }
      this.filterSuggestions();
    },
    addRow() {
      if (this.isValidInput) {
        const selectedMonster = this.suggestions.find(monster => monster.name.toLowerCase() === this.inputValue.toLowerCase());
        this.rows.push(selectedMonster);
        this.guessedMonsters.push(selectedMonster.name.toLowerCase());
        this.inputValue = '';
      } else {
        alert("Please select a monster name from the suggestions, or you have already guessed this monster.");
      }
    },
    filterSuggestions() {
      if (this.inputValue === '') {
        this.filteredSuggestions = [];
        this.showSuggestions = false;
        return;
      }

      this.filteredSuggestions = this.suggestions.filter(suggestion =>
        suggestion.name.toLowerCase().startsWith(this.inputValue.toLowerCase()) && !this.guessedMonsters.includes(suggestion.name.toLowerCase())
      );
      this.showSuggestions = true;
    },
    selectSuggestion(name) {
      this.inputValue = name;
      this.showSuggestions = false;
    },
    formatTypes(types) {
      return Array.isArray(types) ? types.join(', ') : types;
    },
    getRandomMonster() {
      const randomIndex = Math.floor(Math.random() * this.suggestions.length);
      return this.suggestions[randomIndex];
    },
    getCellClass(guessedAttribute, chosenAttribute) {
    // Ensure both are arrays
    if (!Array.isArray(guessedAttribute)) guessedAttribute = [guessedAttribute];
    if (!Array.isArray(chosenAttribute)) chosenAttribute = [chosenAttribute];

    // Sort the arrays
    guessedAttribute.sort((a, b) => (a > b ? 1 : -1));
    chosenAttribute.sort((a, b) => (a > b ? 1 : -1));

    console.log("chosen")
    console.log(chosenAttribute)
    console.log("guess")
    console.log(guessedAttribute)

    if (this.arraysEqual(guessedAttribute, chosenAttribute)) {
        return 'green-background';
    } else if (this.checkIntersection(guessedAttribute, chosenAttribute)) {
        return 'orange-background';
    } else {
        return "";
    }
    },

    arraysEqual(arr1, arr2) {
        if (arr1.length !== arr2.length) return false;
        for (let i = 0; i < arr1.length; i++) {
            if (arr1[i] !== arr2[i]) return false;
        }
        return true;
    },

    checkIntersection(a, b) {
        for (let i = 0; i < a.length; i++) {
            if (b.includes(a[i])) {
                return true;
            }
        }
        return false;
    },



    revealMonster() {
      this.monsterRevealed = true;
    },
    formatMonsterName(name) {
      return name.toLowerCase().replace(/\s+/g, '_');
    },
    getBackgroundStyle(name) {
      const formattedName = this.formatMonsterName(name);
      let imagePath;
      try {
        imagePath = require(`../icons/${formattedName}.png`);
      } catch (error) {
        // Fallback to unknown.png if image for monster doesn't exist
        imagePath = unknownImage;
      }
      
      return {
        backgroundImage: `url(${imagePath})`,
        backgroundSize: 'cover',
        color: 'white',
        textShadow: '2px 2px 5px black', // Adjusted text shadow
        height: this.cellHeight // Set cell height equal to specified height
      };
    },
    newGuess() {
      this.rows = []; // Clear the rows array
      this.guessedMonsters = []; // Reset guessed monsters
      this.monsterRevealed = false; // Reset monster revealed state
      this.randomMonster = this.getRandomMonster(); // Get a new random monster
    }
  }
};
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  margin: 20px;
  background-image: url('../background.jpg');
  background-size: cover;
  background-position: top;
  padding: 20px;
  border-radius: 8px;
  position: relative;
}

.radio-group {
  margin-bottom: 10px;
}

.radio-group label {
  margin-right: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  background-color: transparent;
}

th, td {
  border: 3px solid black;
  padding: 10px;
  text-align: center;
  color: white; /* Set text color to white */
  font-size: 40px; /* Increased font size */
  text-shadow: 
    3px 3px 6px black; /* Increased text shadow */
}

.name-cell {
  width: calc(100% / 11); /* Adjust width for the name column */
}

.table-cell {
  width: calc(100% / 7); /* Equal width for other columns */
  box-sizing: border-box;
  vertical-align: middle;
  height: 200px; /* Set fixed height for other cells */
  overflow: hidden;
  text-overflow: ellipsis;
}

.size-cell {
  position: relative;
}

.green-background {
  background-color: green;
}

.orange-background {
  background-color: orange;
}

.arrow {
  display: inline-block;
  margin-left: 5px;
  width: 0; 
  height: 0; 
  border-left: 15px solid transparent;
  border-right: 15px solid transparent;
}

.arrow.up {
  border-bottom: 30px solid white;
}

.arrow.down {
  border-top: 30px solid white;
}

.monster-cell {
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover; 
}

.size-text {
  display: inline-block;
  max-width: calc(100% - 20px); /* Adjust for arrow width */
  overflow: hidden;
  text-overflow: ellipsis;
}

.suggestions {
  list-style-type: none;
  padding: 0;
  margin: 4px 0 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  position: absolute;
  z-index: 1;
  background-color: #fff;
  box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
}

.suggestions li {
  padding: 10px;
  cursor: pointer;
}

.suggestions li:hover {
  background-color: #f1f1f1;
}

.solve-button {
  position: absolute;
  top: 20px;
  right: 20px;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.solve-button:hover {
  background-color: #0056b3;
}

.new-guess-button {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 10px 20px;
  background-color: #28a745; /* Green color for button */
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.new-guess-button:hover {
  background-color: #218838; /* Darker green on hover */
}
</style>

