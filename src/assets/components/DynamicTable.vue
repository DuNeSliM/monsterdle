<template>
  <div class="container">
    <button class="new-guess-button" @click="newGuess">New Guess</button>
    <div class="radio-group">
      <label>
        <input type="radio" v-model="selectedDataset" value="all"> All Monster
      </label>
      <label>
        <input type="radio" v-model="selectedDataset" value="world"> Monster Hunter World Monster
      </label>
      <label>
        <input type="radio" v-model="selectedDataset" value="generations"> Generations
      </label>
      <div v-if="selectedDataset === 'generations'" class="checkbox-group">
        <label>
          <input type="checkbox" v-model="selectedGenerations" value="gen1"> Generation 1
        </label>
        <label>
          <input type="checkbox" v-model="selectedGenerations" value="gen2"> Generation 2
        </label>
        <label>
          <input type="checkbox" v-model="selectedGenerations" value="gen3"> Generation 3
        </label>
        <label>
          <input type="checkbox" v-model="selectedGenerations" value="gen4"> Generation 4
        </label>
        <label>
          <input type="checkbox" v-model="selectedGenerations" value="gen5"> Generation 5
        </label>
      </div>
    </div>
    <button class="solve-button" @click="revealMonster">Reveal Monster</button>
    <h1>Monsterdle</h1>
    <h2 v-if="monsterRevealed">{{ randomMonster.name }}</h2>
    <label for="inputField">Enter monster name:</label>
    <input v-model="inputValue" type="text" id="inputField" @input="filterSuggestions" @keydown.down.prevent="moveDown" @keydown.up.prevent="moveUp" @keydown.enter.prevent="selectOrSubmit">

    <ul v-if="showSuggestions" class="suggestions">
      <li v-for="(suggestion, index) in filteredSuggestions"
          :key="index"
          :class="{ active: index === activeSuggestion }"
          @click="selectSuggestion(suggestion.name)"
          @mouseover="activeSuggestion = index">
        {{ suggestion.name }}
      </li>
    </ul>

    <button @click="addRow" :disabled="!isValidInput">Add to Table</button>

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

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h3>Congratulations!</h3>
        <p>You guessed the monster in {{ rows.length }} attempts.</p>
        <img :src="getMonsterIcon(randomMonster.name)" alt="Monster Icon" class="monster-icon">
        <p>The monster is: {{ randomMonster.name }}</p>
        <button @click="startNewGame">Start New Game</button>
      </div>
    </div>
  </div>
</template>

<script>
const monstersData = require('./monster.json');
const monstersWorldData = require('./monsterWorld.json');
const gen1Data = require('./gen1.json');
const gen2Data = require('./gen2.json');
const gen3Data = require('./gen3.json');
const gen4Data = require('./gen4.json');
const gen5Data = require('./gen5.json');
const unknownImage = require('../icons/unknown.png');

export default {
  data() {
    return {
      inputValue: '',
      rows: [],
      guessedMonsters: [],
      suggestions: monstersData,
      selectedDataset: 'all', // Default to 'All Monster' dataset
      selectedGenerations: [],
      filteredSuggestions: [],
      showSuggestions: false,
      randomMonster: {},
      monsterRevealed: false,
      cellHeight: '60px', // Adjust as needed for uniform cell height
      showModal: false, // Track if the modal is displayed
      activeSuggestion: -1,
      hasSelectedGeneration: false
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
    },
    selectedGenerations() {
      if (this.selectedDataset === 'generations') {
        this.updateDataset();
        this.newGuess(); // Trigger selection of new random monster when dataset changes
        this.hasSelectedGeneration = this.selectedGenerations.length > 0;
      }
    }
  },
  methods: {

    moveUp() {
      if (this.activeSuggestion > 0) {
        this.activeSuggestion--;
      } else {
        this.activeSuggestion = this.filteredSuggestions.length - 1;
      }
    },
    moveDown() {
      if (this.activeSuggestion < this.filteredSuggestions.length - 1) {
        this.activeSuggestion++;
      } else {
        this.activeSuggestion = 0;
      }
    },
    selectOrSubmit() {
      if (this.activeSuggestion !== -1) {
        const selectedName = this.filteredSuggestions[this.activeSuggestion].name;
        this.selectSuggestion(selectedName);
        this.activeSuggestion = -1;
      } else {
        this.addRow();
      }
    },
    
    updateDataset() {
      if (this.selectedDataset === 'all') {
        this.suggestions = monstersData;
      } else if (this.selectedDataset === 'world') {
        this.suggestions = monstersWorldData;
      } else if (this.selectedDataset === 'generations') {
        this.suggestions = [];
        if (this.selectedGenerations.includes('gen1')) this.suggestions = this.suggestions.concat(gen1Data);
        if (this.selectedGenerations.includes('gen2')) this.suggestions = this.suggestions.concat(gen2Data);
        if (this.selectedGenerations.includes('gen3')) this.suggestions = this.suggestions.concat(gen3Data);
        if (this.selectedGenerations.includes('gen4')) this.suggestions = this.suggestions.concat(gen4Data);
        if (this.selectedGenerations.includes('gen5')) this.suggestions = this.suggestions.concat(gen5Data);
        this.hasSelectedGeneration = this.selectedGenerations.length > 0;
      }
      this.filterSuggestions();
    },
    addRow() {
      if (this.isValidInput) {
        const selectedMonster = this.suggestions.find(monster => monster.name.toLowerCase() === this.inputValue.toLowerCase());
        this.rows.push(selectedMonster);
        this.guessedMonsters.push(selectedMonster.name.toLowerCase());
        this.inputValue = '';
        // Check if the guessed monster is the correct one
        if (selectedMonster.name === this.randomMonster.name) {
          this.showModal = true;
        }
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

    // Convert input value to lowercase for case insensitive comparison
    const inputValueLower = this.inputValue.toLowerCase();

    // Check if input value matches "Cristiano Ronaldo", "Ronaldo", or "suiii" with any number of 'i'
    if (inputValueLower.includes("cristiano ronaldo") || 
        inputValueLower.includes("ronaldo") ||
        /^su+i+$/.test(inputValueLower)) {
      // Suggest "Zinogre" if the input matches
      this.filteredSuggestions = [{ name: "Zinogre" }];
    } else {
      // Otherwise, filter based on default logic
      this.filteredSuggestions = this.suggestions.filter(suggestion =>
        suggestion.name.toLowerCase().startsWith(inputValueLower) && !this.guessedMonsters.includes(suggestion.name.toLowerCase())
      );
    }

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
      if (this.selectedDataset === 'generations' && !this.hasSelectedGeneration) {
        alert("Please select at least one generation.");
        return;
      }
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
    getMonsterIcon(name) {
      const formattedName = this.formatMonsterName(name);
      let imagePath;
      try {
        imagePath = require(`../icons/${formattedName}.png`);
      } catch (error) {
        imagePath = unknownImage;
      }
      return imagePath;
    },
    startNewGame() {
      this.showModal = false;
      this.newGuess();
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
  margin-top: 50px; /* Move radio buttons below New Guess button */
}

.radio-group label {
  margin-right: 20px;
}

.checkbox-group {
  margin-top: 10px; /* Add space above checkboxes */
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

/* Modal styles */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
}

.modal-content h3 {
  margin: 0 0 20px;
}

.modal-content p {
  margin: 10px 0;
}

.monster-icon {
  width: 100px;
  height: 100px;
  margin: 10px 0;
}

.modal-content button {
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 20px;
}

.modal-content button:hover {
  background-color: #218838;
}
</style>
