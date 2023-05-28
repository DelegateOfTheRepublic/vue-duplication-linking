<template>
  <div class="duplication">
    <div class="duplication__type">
      <p>Выберите способ дублирования</p>
      <div class="duplication__type-options">
        <span>
          <input
            id="duplication-type-default"
            type="radio"
            name="duplication-type"
            value="default"
            checked
            v-model="selectedDuplicationType"
          />
          <label for="duplication-type-default">Пара к паре</label>
        </span>
        <span>
          <input
            id="duplication-type-user"
            type="radio"
            name="duplication-type"
            value="user"
            v-model="selectedDuplicationType"
          />
          <label for="duplication-type-user">Пользовательское</label>
        </span>
      </div>
    </div>
    <div class="week-days-tabs" v-if="selectedDuplicationType == 'default'">
      <span
        v-for="dayNumber in week"
        :key="dayNumber"
        @click="activeWeekDayTab.leftSetion = dayNumber"
        :class="[{ 'active-tab': activeWeekDayTab.leftSetion == dayNumber }]"
        >{{ dayNumber }}</span
      >
    </div>
    <div class="wrapper">
      <div class="left-items-with-weekdays">
        <div class="week-days-tabs" v-if="selectedDuplicationType == 'user'">
          <span
            v-for="dayNumber in week"
            :key="dayNumber"
            @click="activeWeekDayTab.leftSetion = dayNumber"
            :class="[
              { 'active-tab': activeWeekDayTab.leftSetion == dayNumber },
            ]"
            >{{ dayNumber }}</span
          >
        </div>
        <div class="left-items">
          <item
            v-for="i in 6"
            :key="i"
            :class="[
              itemSelection(i),
              { 'binding-process': activeLeftItem === i },
            ]"
            @click="setActiveItem(i)"
          />
        </div>
      </div>
      <div class="right-items-with-weekdays">
        <div class="week-days-tabs" v-if="selectedDuplicationType == 'user'">
          <span
            v-for="dayNumber in week"
            :key="dayNumber"
            @click="activeWeekDayTab.rightSection = dayNumber"
            :class="[
              { 'active-tab': activeWeekDayTab.rightSection == dayNumber },
            ]"
            >{{ dayNumber }}</span
          >
        </div>
        <div class="right-items">
          <item
            v-for="i in 6"
            :key="i"
            :class="[isBound(i), alreadyBounded(i)]"
            @click="linkItem(i)"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Item from "./components/Item";
import _ from "lodash";

export default {
  name: "App",
  components: {
    Item,
  },
  data() {
    return {
      week: ["Пн.", "Вт.", "Ср.", "Чт.", "Пт.", "Сб."],
      activateSelection: false,
      activeWeekDayTab: { leftSetion: "Пн.", rightSection: "Пн." },
      activeLeftItem: null,
      linkingItems: {},
      selectedDuplicationType: "default",
    };
  },
  created() {
    this.week.forEach((dayWeek) => (this.linkingItems[dayWeek] = {}));
    console.log();
  },
  methods: {
    setActiveItem(itemId) {
      console.log(123, this.linkingItems);

      if (_.isEqual(this.activeLeftItem, itemId)) {
        if (!this.hasChildren(this.activeLeftItem)) {
          this.activateSelection = !this.activateSelection;
        } else {
          this.activateSelection = false;
        }
        this.activeLeftItem = null;
        return;
      }

      this.activateSelection = true;
      this.activeLeftItem = itemId;
    },
    linkItem(itemId) {
      const isExistItem = this.linkingItems[this.activeWeekDayTab.leftSetion]?.[
        this.activeLeftItem
      ]?.includes(itemId);

      const parentId = this.getParent(itemId);
      console.log(parentId);

      if (!_.isNull(parentId)) {
        const children = this.linkingItems[this.activeWeekDayTab.leftSetion][
          parentId
        ];

        this.linkingItems[this.activeWeekDayTab.leftSetion][
          parentId
        ] = children.filter((childId) => !_.isEqual(childId, itemId));
      }

      if (isExistItem) {
        this.linkingItems[this.activeWeekDayTab.leftSetion][
          this.activeLeftItem
        ] = this.linkingItems[this.activeWeekDayTab.leftSetion][
          this.activeLeftItem
        ].filter((item) => item !== itemId);

        return;
      }

      if (
        _.isEmpty(
          this.linkingItems[this.activeWeekDayTab.leftSetion][
            this.activeLeftItem
          ]
        )
      ) {
        this.linkingItems[this.activeWeekDayTab.leftSetion][
          this.activeLeftItem
        ] = [itemId];
        console.log(this.linkingItems);
        return;
      }

      this.linkingItems[this.activeWeekDayTab.leftSetion][
        this.activeLeftItem
      ].push(itemId);

      console.log(this.linkingItems);
    },
    isBound(itemId) {
      let parentItem = null;

      for (let [key, value] of Object.entries(
        this.linkingItems[this.activeWeekDayTab.leftSetion]
      )) {
        if (value.includes(itemId)) {
          parentItem = key;
        }
      }

      if (this.activateSelection && _.isNull(parentItem)) {
        return "item__to-selection";
      }

      if (!_.isNull(parentItem)) return `item-selection-active-${parentItem}`;

      return;
    },
    hasChildren(parentItemId) {
      return !_.isEmpty(
        this.linkingItems[this.activeWeekDayTab.leftSetion][parentItemId]
      );
    },
    hasParent(childId) {
      for (let [key, value] of Object.entries(
        this.linkingItems[this.activeWeekDayTab.leftSetion]
      )) {
        if (value.includes(childId)) {
          return true;
        }
      }
    },
    getParent(childId) {
      for (let [key, value] of Object.entries(
        this.linkingItems[this.activeWeekDayTab.leftSetion]
      )) {
        if (value.includes(childId)) {
          return key;
        }
      }

      return null;
    },
    isParent(childId, parentId) {
      return this.linkingItems[this.activeWeekDayTab.leftSetion][
        parentId
      ]?.includes(childId);
    },
    itemSelection(itemId) {
      if (this.activeLeftItem === itemId || this.hasChildren(itemId))
        return `item-selection-active-${itemId}`;

      return `item-selection-${itemId}`;
    },
    alreadyBounded(childId) {
      if (this.hasParent(childId) && this.activeLeftItem) {
        return ["already-has-parent", `item-selection-${this.activeLeftItem}`];
      }
    },
  },
  computed: {},
};
</script>

<style scoped>
.duplication {
}

.duplication__type {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  margin-bottom: 30px;
}

.duplication__type-options {
  display: flex;
  width: fit-content;
  gap: 12px;
}

.week-days-tabs {
  margin-bottom: 35px;
}

.week-days-tabs > span {
  background: rgb(110, 255, 81);
  outline: 0px solid rgb(81, 142, 255);
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
  transition: outline 0.05s linear;
}

.week-days-tabs > span:not(last-of-type) {
  margin-right: 25px;
}

.active-tab {
  outline: 2px solid rgb(81, 142, 255) !important;
}

.week-days-tabs > span:hover {
  outline: 2px solid rgb(81, 142, 255);
}

.left-items-with-weekdays,
.right-items-with-weekdays {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.wrapper {
  display: flex;
  justify-content: space-between;
  padding: 0 50px;
}

.left-items > .item {
  cursor: pointer;
}

.already-has-parent {
  transition: all 0.15s ease;
}

.already-has-parent:hover {
  cursor: pointer;
  outline-style: dashed !important;
  outline-offset: 4px;
}

.item-selection-active-1,
.item-selection-1:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-1));
}

.item-selection-active-2,
.item-selection-2:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-2));
}

.item-selection-active-3,
.item-selection-3:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-3));
}

.item-selection-active-4,
.item-selection-4:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-4));
}

.item-selection-active-5,
.item-selection-5:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-5));
}

.item-selection-active-6,
.item-selection-6:hover {
  outline: var(--item-selection-base) rgba(var(--item-selection-color-6));
}

.binding-process {
  animation: fading 2s ease 0s infinite normal forwards;
}

.left-items,
.right-items {
  padding: 15px;
  overflow-y: auto;
  overflow-x: hidden;
  height: 700px;
  background: rgb(110, 255, 81);
}

.left-items > .item:not(:first-of-type),
.right-items > .item:not(:first-of-type) {
  margin-top: 50px;
}

@keyframes fading {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(0.98);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes glowing {
  0% {
    background-color: cornflowerblue;
    box-shadow: 0 0 5px cornflowerblue;
  }
  50% {
    background-color: rgb(42, 109, 235);
    box-shadow: 0 0 20px rgb(42, 109, 235);
  }
  100% {
    background-color: cornflowerblue;
    box-shadow: 0 0 5px cornflowerblue;
  }
}

.item__to-selection {
  animation: glowing 5000ms infinite;
  cursor: pointer;
}
</style>

<style>
:root {
  --item-selection-base: 3px solid;
  --item-selection-color-alpha: 1;
  --item-selection-color-1: 68, 0, 255;
  --item-selection-color-2: 255, 0, 98;
  --item-selection-color-3: 0, 119, 255;
  --item-selection-color-4: 136, 32, 255;
  --item-selection-color-5: 218, 30, 255;
  --item-selection-color-6: 255, 205, 25;
}

P {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
