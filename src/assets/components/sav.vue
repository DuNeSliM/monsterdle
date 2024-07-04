<template>
    <div class="container">
      <h1>Dynamic Table Update</h1>
      <label for="inputField">Enter monster name:</label>
      <input v-model="inputValue" type="text" id="inputField" @input="filterSuggestions">
      <ul v-if="showSuggestions" class="suggestions">
        <li v-for="(suggestion, index) in filteredSuggestions" :key="index" @click="selectSuggestion(suggestion.name)">{{ suggestion.name }}</li>
      </ul>
      <button @click="addRow" :disabled="!isValidInput">Add to Table</button>
  
      <table>
        <thead>
          <tr>
            <th>Monster Name</th>
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
            <td>{{ row.name }}</td>
            <td>{{ row.type }}</td>
            <td>{{ formatTypes(row.elements) }}</td>
            <td>{{ formatTypes(row.weaknesses) }}</td>
            <td>{{ row.size }}</td>
            <td>{{ formatTypes(row.iconColor) }}</td>
            <td>{{ row.firstAppearance }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script>
  const monstersData = require('./monster.json');
  
  export default {
    data() {
      return {
        inputValue: '',
        rows: [],
        suggestions: monstersData,
        filteredSuggestions: [],
        showSuggestions: false
      };
    },
    computed: {
      isValidInput() {
        return this.filteredSuggestions.some(suggestion => suggestion.name.toLowerCase() === this.inputValue.toLowerCase());
      }
    },
    methods: {
      addRow() {
        if (this.isValidInput) {
          const selectedMonster = this.suggestions.find(monster => monster.name.toLowerCase() === this.inputValue.toLowerCase());
          this.rows.push(selectedMonster);
          this.inputValue = '';
        } else {
          alert("Please select a monster name from the suggestions.");
        }
      },
      filterSuggestions() {
        if (this.inputValue === '') {
          this.filteredSuggestions = [];
          this.showSuggestions = false;
          return;
        }
  
        this.filteredSuggestions = this.suggestions.filter(suggestion =>
          suggestion.name.toLowerCase().startsWith(this.inputValue.toLowerCase())
        );
        this.showSuggestions = true;
      },
      selectSuggestion(name) {
        this.inputValue = name;
        this.showSuggestions = false;
      },
      formatTypes(types) {
      return Array.isArray(types) ? types.join(', ') : types;
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
  text-align: left;
  color: white; /* Set text color to white */
  text-shadow: 
    -1px -1px 0 #000,  
    1px -1px 0 #000,
    -1px 1px 0 #000,
    1px 1px 0 #000; /* Outline text with black */
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
  </style>
  