<template>
  <div class="board">
    <!-- Main - Head Section Start -->
    <div class="header-wrapper">
      <h1 class="main-text">{{ msg }}</h1>
      <button class="add-list-btn" @click="addNewList()">Add New List</button>
    </div>
    <!-- Main - Head Section End -->

    <!-- Tre-yelo list Section Begin -->
    <div class="list-wrapper">
      <div
        class="vertical-list-container"
        v-bind:key="index"
        v-for="(list, index) in lists"
        @drop="drop"
        @dragenter.prevent
        @dragover.prevent
        :id="`dropTarget-${index}`"
      >
        <div class="remove-btn" @click="removeList(index)">x</div>
        <div
          class="list-title ellipsis100"
          @input="nameChange($event, index, null, 'list')"
          contenteditable="true"
        >
          {{ list.title }}
        </div>
        <!-- Tre-yelo Card Section Begin -->
        <div
          class="card"
          v-bind:key="i"
          v-for="(card, i) in list.cards"
          @dragstart="dragStart"
          draggable="true"
          :id="`dragTarget-${index}-${i}`"
        >
          <div class="remove-btn" @click="removeCard(index, i)">x</div>
          <div
            class="card-title ellipsis100"
            @input="nameChange($event, index, i, 'cardTitle')"
            contenteditable="true"
          >
            {{ card.title }}
          </div>
          <div
            class="card-description ellipsis100"
            @input="nameChange($event, index, i, 'cardDescription')"
            contenteditable="true"
          >
            {{ card.description }}
          </div>
        </div>
        <!-- Tre-yelo Card Section End -->
        <div class="add-btn" @click="addNewCard(index)">+</div>
      </div>
    </div>
    <!-- Tre-yelo List Section End -->
  </div>
</template>

<script>
import List from "../assets/list.json";
export default {
  name: "Board",
  props: {
    msg: String,
  },
  methods: {
    initList() {
      // Initialize lists if no data present in storage
      return List.data;
    },
    dragStart(event) {
      // Remember the element being dragged
      event.dataTransfer.setData("ListItem", event.target.id);

      // Hide all the add buttons in list
      this.dragAction("hide");
    },
    dragAction(action) {
      // Select all add buttons in list
      let btns = document.querySelectorAll(".add-btn");
      for (let i = 0; i < btns.length; i++) {
        // Hide button if some card is being dragged
        btns[i].style.display = action === "hide" ? "none" : "initial";
      }
    },
    drop(event) {
      event.preventDefault();
      // Read item last dragged
      let data = event.dataTransfer.getData("ListItem");

      // List Index
      let fromListIndex = data.split("-")[1];
      // Card Position
      let fromListCardIndex = data.split("-")[2];
      // List Index that the card was dropped onto
      let targetListIndex = event.target.id.split("-")[1];
      if (targetListIndex !== fromListIndex) {
        // unshift the new card
        // Only if target list is DIFF as source list
        this.lists[targetListIndex].cards.unshift(
          this.lists[fromListIndex].cards[fromListCardIndex]
        );
        // Remove the card from original list
        this.removeCard(fromListIndex, fromListCardIndex);
      } else {
        // If target list is same as source list
        // do not unshift the new card
        event.target.appendChild(document.getElementById(data));
      }

      // Show the add button after card is dragged
      this.dragAction("show");
    },
    addNewList() {
      this.lists.push({ title: `List ${this.lists.length}`, cards: [] });
    },
    addNewCard(listIndex) {
      this.lists[listIndex].cards.push({
        title: `Card [${listIndex}][${this.lists[listIndex].cards.length}]`,
        description: `Card of ${this.lists[listIndex].title}`,
      });
    },
    nameChange(e, listIndex, cardIndex, obj) {
      if (obj === "list") {
        this.lists[listIndex].title = e.target.innerHTML;
      } else if (obj === "cardTitle") {
        this.lists[listIndex].cards[cardIndex].title = e.target.innerHTML;
      } else if (obj === "cardDescription") {
        this.lists[listIndex].cards[cardIndex].description = e.target.innerHTML;
      }
    },
    removeCard(listIndex, cardIndex) {
      this.lists[listIndex].cards.splice(cardIndex, 1);
    },
    removeList(listIndex) {
      this.lists.splice(listIndex, 1);
    },
    updateBoard() {
      localStorage.setItem("lists", JSON.stringify(this.lists));
    },
    getLists() {
      return JSON.parse(localStorage.getItem("lists"));
    },
  },
  data() {
    return {
      lists: [],
    };
  },
  beforeUpdate() {
    this.updateBoard();
  },
  mounted() {
    this.lists = this.listsArray;
  },
  computed: {
    listsArray() {
      if (this.getLists() == null || !this.getLists().length) {
        return this.initList();
      } else {
        return this.getLists();
      }
    },
  },
};
</script>

<style scoped>
.header-wrapper {
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.main-text {
  color: #2c3e50;
}
.board {
  position: relative;
}
.list-wrapper {
  overflow-x: scroll;
  display: block;
  white-space: nowrap;
}
.vertical-list-container {
  max-width: 100px;
  width: 100px;
  overflow-y: scroll;
  height: 100vh;
  max-height: 70vh;
  background: #e4e4e4;
  position: relative;
  padding: 5px;
  display: inline-block;
  margin: 10px 2px;
  border-radius: 3px;
}
.list-title {
  margin: 10px;
  font-size: 16px;
  font-weight: 900;
}
.remove-btn {
  position: absolute;
  right: 10px;
  font-size: 10px;
  border-radius: 50%;
  width: 15px;
  height: 15px;
  padding: 0;
  cursor: pointer;
  color: #6f6f6f;
}
.remove-btn:hover {
  background: #d0d0d0;
  transition: background 0.5s;
}
.add-btn {
  position: sticky;
  bottom: 0;
  height: 20px;
  width: 20px;
  border-radius: 50px;
  cursor: pointer;
  background: #ddd;
  margin: 0 auto;
  margin-top: 20px;
}
.add-list-btn {
  background: linear-gradient(
    90deg,
    hsla(350, 93%, 61%, 1) 0%,
    hsla(8, 98%, 59%, 1) 100%
  );
  color: #fff;
  font-weight: 600;
  padding: 10px 20px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  box-shadow: 0px 2px 5px 1px rgba(53, 53, 53, 0.281);
}
.add-list-btn:active {
  box-shadow: none;
  transition: box-shadow 0.3s;
}
.card {
  width: 100%;
  max-height: 100px;
  overflow: scroll;
  background: white;
  margin-bottom: 5px;
  cursor: pointer;
  border-radius: 3px;
}
.card-title {
  font-size: 14px;
  font-weight: 700;
  margin: 10px 0;
}
.card-description {
  font-size: 12px;
  margin: 10px 0;
}
.ellipsis100 {
  max-width: 100px;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
