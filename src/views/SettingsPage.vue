<template>
  <div style="padding: 3.5rem 0 0 1rem;flex-grow: 1;overflow: auto;position: relative;">
    <section v-if="show">
      <transition name="fade" appear>
        <div
          class="card-header"
          :style="{
            backgroundColor:
              message === message
                ? 'rgb(105, 222, 64)'
                : '#cd5c5c',
          }"
        >
          <span>{{ message }}</span>
          <div @click="show = false" class="cross">✕</div>
        </div>
      </transition>
    </section>
    <!-- TABS -->
    <div class="tabs">
      <button @click="activeTab = 'synonyms'" :class="{ 'active': activeTab === 'synonyms' }">Sinonimi i pragovi</button>
      <button @click="activeTab = 'subjects'" :class="{ 'active': activeTab === 'subjects' }">Lista tema</button>
      <button @click="activeTab = 'live'" :class="{ 'active': activeTab === 'live' }">Live</button>
      <!-- Add more buttons for additional tabs -->
    </div>

      <!-- SYNONYMS TAB CONTENT -->
      <div v-if="activeTab === 'synonyms'" class="tabcontent">
        <h3>Popis sinonima</h3>
        <div v-for="(new_value, index) in newValues" :key="index" class="synonym-group">
          {{ index + 1 + ". " }}
          <div class="synonym-info">
            <div class="input-container">
              <div contenteditable="true" @blur="handleBlurKey($event.target.textContent, index)" placeholder="Prazno polje" aria-describedby="" maxlength="128" class="custom-input">{{ new_value }}</div>
              <div class="icon-container">
                <svg class="bi-pencil" viewBox="0 0 192 192" xmlns="http://www.w3.org/2000/svg" xml:space="preserve" fill="none"> <g id="SVGRepo_bgCarrier" stroke-width="0"></g> <g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="m104.175 90.97-4.252 38.384 38.383-4.252L247.923 15.427V2.497L226.78-18.646h-12.93zm98.164-96.96 31.671 31.67" class="cls-1" style=" fill: none; fill-opacity: 1; fill-rule: nonzero; stroke: #000000; stroke-width: 12; stroke-linecap: round; stroke-linejoin: round; stroke-dasharray: none; stroke-opacity: 1; " transform="translate(-77.923 40.646)" ></path> <path d="m195.656 33.271-52.882 52.882" style=" fill: none; fill-opacity: 1; fill-rule: nonzero; stroke: #000000; stroke-width: 12; stroke-linecap: round; stroke-linejoin: round; stroke-miterlimit: 5; stroke-dasharray: none; stroke-opacity: 1; " transform="translate(-77.923 40.646)" ></path> </g> </svg>
              </div>
            </div>
            &larr;
            <div class="old-values">
            <div contenteditable="true" @blur="handleBlurValue($event, index)" style="min-width:115px; background: var(--background); padding-right: 2rem; scroll-margin-bottom: 2rem; width: 100%; padding: 10px; line-height: 17px; box-sizing: border-box; word-wrap: break-word;">{{ editedOldValues[index] }}</div>
          </div>
          <div>
            <button @click="deleteSynonyms(index)" style="align-items: center; display: inline-flex;">
              <svg style="margin-top: 6px" focusable="false" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg" fill="currentColor" aria-label="Close" aria-hidden="true" width="16" height="16" viewBox="0 0 32 32" role="img" class="bx--btn__icon"> <path d="M24 9.4L22.6 8 16 14.6 9.4 8 8 9.4 14.6 16 8 22.6 9.4 24 16 17.4 22.6 24 24 22.6 17.4 16 24 9.4z"></path></svg>
              </button>
            </div>
          </div>
        </div>
        <button class="color-button" @click="addSynonymGroup">Dodaj novu grupu sinonima +</button>
        <h3>Pragovi</h3>
        <div>
          <div style="align-items: center; display: flex; flex-wrap: wrap;">
            <span>1. Unesite gornji prag za koji chatbot direktno vraća odgovor:</span>
            <div contenteditable="true" @blur="upperThreshold = $event.target.innerText" style="background: var(--background); padding-right: 2rem; scroll-margin-bottom: 2rem; padding: 8px 15px 8px 15px; line-height: 17px; box-sizing: border-box; word-wrap: break-word; margin: 0px 7px 0px 7px;">{{ upperThreshold }}</div>
            <p style="font-size: 9px;">(0.0 - 100.0)</p>
          </div>
          <div style="align-items: center; flex-wrap: wrap; display: flex; flex-wrap: wrap;">
            <span>2. Unesite donji prag za koji chatbot traži korisnika da ponovno pita pitanje:</span>
            <div contenteditable="true" @blur="lowerThreshold = $event.target.innerText" style="background: var(--background); padding-right: 2rem; scroll-margin-bottom: 2rem; padding: 8px 15px 8px 15px; line-height: 17px; box-sizing: border-box; word-wrap: break-word; margin: 0px 7px 0px 7px;">{{ lowerThreshold }}</div>
            <p style="font-size: 9px;">(0.0 - 100.0)</p>
          </div>
        </div>
        <button @click="saveSynonyms" class="background-button" tabindex="0" type="button" style="margin-top: 2rem;">Spremi sinonime i pragove
          <svg class="svg" focusable="false" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg" fill="currentColor" aria-label="Saved" aria-hidden="true" width="16" height="16" viewBox="0 0 16 16" role="img"><path d="M13.9,4.6l-2.5-2.5C11.3,2.1,11.1,2,11,2H3C2.4,2,2,2.4,2,3v10c0,0.6,0.4,1,1,1h10c0.6,0,1-0.4,1-1V5	C14,4.9,13.9,4.7,13.9,4.6z M6,3h4v2H6V3z M10,13H6V9h4V13z M11,13V9c0-0.6-0.4-1-1-1H6C5.4,8,5,8.4,5,9v4H3V3h2v2c0,0.6,0.4,1,1,1	h4c0.6,0,1-0.4,1-1V3.2l2,2V13H11z"></path></svg>
        </button>

      </div>

      <!-- Subjects TAB CONTENT -->
      <div v-if="activeTab === 'subjects'" class="tabcontent">
        <h3>Popis početnih tema za chatbot</h3>
        <div class="input-control" style="width: 20vw">
          <label>Pretraži sve namjere</label>
          <div class="custom-checkbox-select" tabindex="1">
            <div class="selected-items" @click="toggleDropdown" ref="selectedItemsSettingsRef">
              <span v-if="selectedIntents.length === 0" style="overflow: hidden;white-space: nowrap;text-overflow: ellipsis;color: #757575;min-width: 1rem;">Odaberite namjere...</span>
              <span style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-width: 1rem;" v-else>
                {{ selectedIntents.map(intent => intent.name).join(", ") }}
              </span>
              <svg class="dropdown-arrow" viewBox="0 0 16 16"><path d="M8 11L3 6 3.7 5.3 8 9.6 12.3 5.3 13 6z"></path><title>Open menu</title></svg>
            </div>
            <div v-show="isDropdownOpen" class="dropdown" ref="dropdownSettingsRef">
              <button style="display: block; padding: 8px; color: var(--main__color)" @click="removeChosenIntents">Ukloni sve...</button>
              <label v-for="intent in uniqueIntents" :key="intent.key" class="checkbox-label">
                <input
                  type="checkbox"
                  :value="intent"
                  v-model="selectedIntents"
                />
                {{ intent.name }}
              </label>
            </div>
          </div>
        </div>
        <div style="padding: 1rem 0 0 0.5rem;">
          <h4>Odabrane namjere:</h4>
          <div v-for="(intent, index) in selectedIntents" :key="index">
            <p>{{ index + 1 }}. {{ intent.name }}</p>
          </div>
        </div>
        <button @click="saveSubjects" class="background-button" tabindex="0" type="button" style="margin-top: 2rem;">Spremi teme
          <svg class="svg" focusable="false" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg" fill="currentColor" aria-label="Saved" aria-hidden="true" width="16" height="16" viewBox="0 0 16 16" role="img"><path d="M13.9,4.6l-2.5-2.5C11.3,2.1,11.1,2,11,2H3C2.4,2,2,2.4,2,3v10c0,0.6,0.4,1,1,1h10c0.6,0,1-0.4,1-1V5	C14,4.9,13.9,4.7,13.9,4.6z M6,3h4v2H6V3z M10,13H6V9h4V13z M11,13V9c0-0.6-0.4-1-1-1H6C5.4,8,5,8.4,5,9v4H3V3h2v2c0,0.6,0.4,1,1,1	h4c0.6,0,1-0.4,1-1V3.2l2,2V13H11z"></path></svg>
        </button>
      </div>

      <!-- LIVE TAB CONTENT -->
      <div v-if="activeTab === 'live'" class="tabcontent">
        <h3>Ažuriranje sustava</h3>
        <div style="display:flex"><p style="font-style:italic">Posljednje ažuriranje sustava</p>: <p style="text-decoration:underline;margin-left:10px">7.siječnja 2024.</p></div>
        <button
          @click="live"
          class="background-button"
          tabindex="0"
          type="button"
          style="margin-top: 2rem;color:white;background:black"
        >
          Verzioniranje sustava
          <svg class="svg" viewBox="-0.5 0 25 25" style="fill:white" height="25px" width="25px" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M18.8887 10.25C18.5395 10.2462 18.1974 10.151 17.8964 9.97387C17.5954 9.79677 17.3461 9.54393 17.1731 9.24053C17.0002 8.93714 16.9097 8.59372 16.9107 8.2445C16.9117 7.89528 17.0041 7.55241 17.1787 7.24999L17.2987 7.03997C17.4297 6.81781 17.5133 6.57097 17.5443 6.31493C17.5753 6.0589 17.553 5.79924 17.4788 5.55224C17.4046 5.30524 17.2801 5.07626 17.1132 4.87969C16.9462 4.68312 16.7404 4.52317 16.5087 4.40996V4.40996C16.0227 4.17652 15.467 4.13332 14.9507 4.28875C14.4345 4.44418 13.995 4.78704 13.7187 5.24999V5.24999C13.5404 5.54389 13.2894 5.78686 12.9899 5.9555C12.6903 6.12413 12.3524 6.21276 12.0087 6.21276C11.665 6.21276 11.327 6.12413 11.0275 5.9555C10.728 5.78686 10.4769 5.54389 10.2987 5.24999C10.0224 4.78704 9.58291 4.44418 9.06665 4.28875C8.5504 4.13332 7.99469 4.17652 7.5087 4.40996V4.40996C7.27697 4.52317 7.07116 4.68312 6.90421 4.87969C6.73726 5.07626 6.61277 5.30524 6.53858 5.55224C6.46438 5.79924 6.44209 6.0589 6.47309 6.31493C6.50408 6.57097 6.5877 6.81781 6.71869 7.03997L6.83869 7.24999C7.01332 7.55241 7.10571 7.89528 7.10669 8.2445C7.10767 8.59372 7.01721 8.93714 6.84427 9.24053C6.67134 9.54393 6.42196 9.79677 6.12097 9.97387C5.81999 10.151 5.4779 10.2462 5.12869 10.25C4.64217 10.238 4.16698 10.3979 3.78659 10.7015C3.40621 11.005 3.14493 11.4329 3.04868 11.91C2.99868 12.1996 3.01314 12.4967 3.09101 12.7801C3.16887 13.0635 3.30826 13.3263 3.49921 13.5497C3.69016 13.7731 3.92799 13.9516 4.1958 14.0727C4.46362 14.1937 4.75481 14.2543 5.04868 14.25H5.1687C5.5179 14.2538 5.86 14.349 6.16098 14.5261C6.46196 14.7032 6.71131 14.9561 6.88425 15.2595C7.05718 15.5628 7.14768 15.9063 7.1467 16.2555C7.14572 16.6047 7.05333 16.9476 6.87869 17.25L6.82868 17.33C6.56685 17.7935 6.4956 18.3407 6.62998 18.8558C6.76435 19.3709 7.0938 19.8135 7.54868 20.09V20.09C8.00218 20.351 8.53992 20.4239 9.04654 20.293C9.55316 20.1622 9.98834 19.838 10.2587 19.39L10.2787 19.25C10.457 18.9561 10.708 18.7131 11.0075 18.5445C11.307 18.3759 11.6449 18.2872 11.9887 18.2872C12.3324 18.2872 12.6704 18.3759 12.9699 18.5445C13.2694 18.7131 13.5204 18.9561 13.6987 19.25L13.7687 19.39C14.0391 19.8407 14.4761 20.1668 14.9851 20.2978C15.4942 20.4288 16.0343 20.3542 16.4887 20.09C16.9367 19.8197 17.2609 19.3845 17.3917 18.8779C17.5226 18.3712 17.4497 17.8335 17.1887 17.38L17.1287 17.27C16.9541 16.9676 16.8617 16.6247 16.8607 16.2754C16.8597 15.9262 16.9502 15.5829 17.1231 15.2795C17.296 14.9761 17.5454 14.7232 17.8464 14.5461C18.1474 14.369 18.4895 14.2738 18.8387 14.27H18.9587C19.2525 14.2743 19.5438 14.2138 19.8116 14.0927C20.0794 13.9717 20.3172 13.793 20.5082 13.5696C20.6991 13.3462 20.8385 13.0835 20.9164 12.8001C20.9942 12.5167 21.0087 12.2196 20.9587 11.93C20.8669 11.451 20.6088 11.0198 20.2301 10.7124C19.8514 10.405 19.3763 10.2413 18.8887 10.25V10.25Z" stroke="#000000" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M12 14.79C13.3807 14.79 14.5 13.6707 14.5 12.29C14.5 10.9093 13.3807 9.78998 12 9.78998C10.6193 9.78998 9.5 10.9093 9.5 12.29C9.5 13.6707 10.6193 14.79 12 14.79Z" stroke="#000000" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path> </g></svg>
        </button>
      </div>

      <!-- Add more divs for additional tab contents -->
    </div>

    <!-- Grey background at the bottom -->
    <div class="bottom-section"/>
</template>
<script>
import DataService from "../services/data.services";
export default {
  data() {
    return {
      editing: false,
      newValues: [],
      oldValuesMap: new Map(),
      editedOldValues: [],
      isEditable: [],
      upperThreshold: null,
      lowerThreshold: null,
      show: false,
      message: "",
      activeTab: 'synonyms', // Set the default active tab
      uniqueIntents: [],
      isDropdownOpen: false,
      selectedIntents: [],
    };
  },
  async created() {
    this.filterOldValues();
    this.getIntents();
    try {
      let res = await DataService.getThresholdsBySystemId(
        this.$route.query.system_id
      );
      this.upperThreshold = res[0].percentage_upper;
      this.lowerThreshold = res[0].percentage_lower;
    } catch (error) {
      console.error("Error fetching thresholds:", error);
    }
  },
  methods: {
    async filterOldValues() {
      try {
        let synonyms = await DataService.getSynonyms(
          this.$route.query.system_id
        );

        if (Array.isArray(synonyms) && synonyms.length > 0) {
          const synonymMap = new Map();

          synonyms.forEach((item) => {
            const { new_value, old_value } = item;

            if (!synonymMap.has(new_value)) {
              synonymMap.set(new_value, [old_value]);
            } else {
              synonymMap.get(new_value).push(old_value);
            }
          });

          this.newValues = Array.from(synonymMap.keys());
          this.oldValuesMap = synonymMap;
          this.editedOldValues = this.newValues.map((value) =>
            this.oldValuesMap.get(value).join(", ")
          );
          this.isEditable = new Array(this.newValues.length).fill(false);
        } else {
          this.newValues = [];
          this.oldValuesMap = new Map();
          this.editedOldValues = [];
          this.isEditable = [];
        }
      } catch (error) {
        console.error("Error fetching synonyms:", error);
      }
    },
    async getIntents() {
      let objects = await DataService.getIntentsForSystem(this.$route.query.system_id);
      this.uniqueIntents = objects.map(element => ({ name: element.name, key: element.id }));
      this.uniqueIntents.sort((a, b) => a.name.localeCompare(b.name));
      objects = await DataService.getThemes(this.$route.query.system_id)
      objects[0].intents ? this.selectedIntents = JSON.parse(objects[0].intents) : this.selectedIntents = []
    },
    deleteSynonyms(index) {
      const newValueToDelete = this.newValues[index];
      this.newValues.splice(index, 1);
      this.oldValuesMap.delete(newValueToDelete);
      this.editedOldValues.splice(index, 1);
      this.isEditable.splice(index, 1);
    },
    addSynonymGroup() {
      this.newValues.push("New Value");
      this.oldValuesMap.set("New Value", ["Old Value"]);
      this.editedOldValues.push("");
      this.isEditable.push(true);
    },
    handleBlurKey(value, index) {
      // Handle the input on blur
      const oldKey = this.newValues[index];
      const oldValueArray = this.oldValuesMap.get(oldKey) || [];

      // Remove the old key-value pair
      this.oldValuesMap.delete(oldKey);

      // Set the new key and value in the map
      this.oldValuesMap.set(value, oldValueArray);
      this.newValues[index] = value;
    },

    handleBlurValue(event, index) {
      let [key, values] = Array.from(this.oldValuesMap.entries())[index];
      if (key !== undefined && values !== undefined) {
        this.oldValuesMap.delete(key);
        const newValues = event.target.innerText
          .split(",")
          .map((value) => value.trim());
        this.oldValuesMap.set(key, newValues);
      }
    },
    async saveSynonyms() {
      try {
        const synonymsObject = {};

        this.oldValuesMap.forEach((value, key) => {
          synonymsObject[key] = value;
        });

        await DataService.updateSynonyms(
          this.$route.query.system_id,
          synonymsObject
        );
        await DataService.updateThresholdsBySystemId(
          this.$route.query.system_id,
          this.upperThreshold,
          this.lowerThreshold
        );
        this.show = true;
        this.message = "Uspješno spremljene promjene.";
        setTimeout(() => {
          this.show = false;
        }, 4000);
      } catch (error) {
        this.show = true;
        this.message =
          "Pogreška prilikom spremanja promjena. Molim Vas pokušajte ponovno.";
        setTimeout(() => {
          this.show = false;
        }, 4000);
      }
    },
    toggleDropdown() {
      this.isDropdownOpen = !this.isDropdownOpen;
      if (this.isDropdownOpen) {
        document.addEventListener("click", this.closeDropdownOnOutsideClick);
      } else {
        document.removeEventListener("click", this.closeDropdownOnOutsideClick);
      }
    },
    closeDropdownOnOutsideClick(event) {
      const dropdown = this.$refs.dropdownSettingsRef;
      const selectedItems = this.$refs.selectedItemsSettingsRef;
      if (dropdown && selectedItems) {
        if (
          !dropdown.contains(event.target) &&
          !selectedItems.contains(event.target)
        ) {
          this.isDropdownOpen = false;
          document.removeEventListener("click", this.closeDropdownOnOutsideClick);
        }
      }
    },
    removeChosenIntents() {
      this.selectedIntents = [];
    },
    async saveSubjects(){
      try{
        await DataService.updateThemes(this.$route.query.system_id, this.selectedIntents)
        this.show = true;
        this.message = "Uspješno spremljene promjene.";
        setTimeout(() => {
          this.show = false;
        }, 4000);
      }
      catch (error) {
        this.show = true;
        this.message =
          "Pogreška prilikom spremanja promjena. Molim Vas pokušajte ponovno.";
        setTimeout(() => {
          this.show = false;
        }, 4000);
      }
    },
  },
};
</script>

<style scoped>
.synonym-group {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.synonym-info {
  align-items: center;
  display: flex;
  justify-content: space-between;
}

.new-value {
  font-weight: bold;
  cursor: pointer;
}

.editable {
  background-color: #f0f8ff; /* Light blue background for editable state */
}

.old-values {
  flex-grow: 1;
  margin-left: 10px;
}

button {
  margin-left: 10px;
}

.input-container {
  display: flex;
  flex: 0 0 13%;
  margin: 2px;
  align-items: center;
  border-bottom: 1px solid #8d8d8d;
  position: relative;
  height: 4vh;
}

.custom-input {
  height: 90%;
  display: flex;
  align-items: center;
  font-weight: 800;
  flex-grow: 1;
  padding-right: 2rem;
  padding-left: 2.5rem;
  min-width: 100px;
}

.icon-container {
  width: 2.5rem; /* Width of the icon container */
  display: flex;
}

/* Style the SVG icon as needed */
.bi-pencil {
  width: 16px; /* Icon width */
  height: 16px; /* Icon height */
  background-repeat: no-repeat;
  background-size: contain;
  cursor: pointer; /* Add a cursor pointer for interaction */
}

.svg {
  height: 1rem;
  position: absolute;
  right: 1rem;
  width: 1rem;
  margin-right: 4px;
  margin-top: 2px;
}

/*popup message*/
.card-header {
  height: 20px;
  vertical-align: middle;
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
  line-height: 20px;
  padding: 15px;
  color: white;
  position: absolute;
  top: 1rem;
  right: 1rem;
  min-width: 420px;
  border-radius: 5px;
  padding-right: 5rem;
  box-shadow: 0 0 15px 5px #ccc;
}

.cross {
  position: absolute;
  top: 16px;
  right: 15px;
  font-size: 25px;
  font-weight: 100;
  cursor: pointer;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s linear;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/*tabs style*/
.tabs {
  display: flex;
  margin-bottom: 20px;
}

.tabs button {
  min-width: 13rem;
  text-align: left;
  padding: 0.7rem 1rem;
  margin-right: 10px;
  cursor: pointer;
  background: var(--background);
  border: 1px solid #ccc;
}

.tabs button.active {
  font-weight: bold;
  filter: brightness(85%);
}

.tabcontent{
  -webkit-animation: fadeEffect 1s;
  animation: fadeEffect 1s;
  padding: 1rem 1rem 1rem 2rem;
  box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  margin: 0 1rem;
}

/* Fade in tabs */
@-webkit-keyframes fadeEffect {
  from {opacity: 0;}
  to {opacity: 1;}
}

@keyframes fadeEffect {
  from {opacity: 0;}
  to {opacity: 1;}
}
</style>
