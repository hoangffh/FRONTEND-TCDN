<template>
  <div
    :class="['base-multi-select', { disable: isDisable }]"
    ref="input_wrap"
    :style="{ width: width, minHeight: `${minHeight}px` }"
  >
    <div class="input__wrap">
      <div
        class="multi__box__checked__item"
        v-for="(option, index) in listOptionSelected"
        :key="index"
      >
        <div
          class="multi__box__checked__item__icon"
          @click="handleClickRemoveTag(option)"
        ></div>
        {{ handleDisplayOptionSelected(option) }}
      </div>
      <input
        class="cbb__input"
        @keydown="handleKeyDownInput"
        @input="handleOnInput"
        :style="{ height: `${minHeightHandled}px`, width: width }"
        :tabindex="tabindex"
        v-model="textSearch"
        :disabled="isDisable"
      />
    </div>
    <div class="multi__select__action">
      <div class="cbb__icon__add-wrap" @click="handleClickAdd">
        <div class="cbb__icon__add"></div>
      </div>
      <div class="cbb__icon-wrap" @click="handleClickDisplayIcon">
        <div class="cbb__icon-hide" v-show="!isShowList"></div>
        <div class="cbb__icon-show" v-show="isShowList"></div>
      </div>
    </div>
  </div>
  <div
    class="multi_cbb__list"
    v-show="isShowList"
    :style="{
      top: `${dropDownY}px`,
      left: `${dropDownX}px`,
      minWidth: `${inputWidth}px`,
    }"
  >
    <table :style="{ width: '100%' }">
      <thead class="cbb__list__item-header">
        <tr>
          <th
            class="cbb__item__col"
            v-for="(item, index) in infor"
            :key="index"
          >
            <div :style="{ width: item.width }">
              {{ item.display }}
            </div>
          </th>
        </tr>
      </thead>
      <tbody ref="listCbb">
        <tr
          v-for="(dataItem, dataIndex) in dataChanged"
          :key="dataIndex"
          @click="handleClickOption(dataItem)"
          :class="[
            `multiSelectItem-${dataIndex}`,
            modelValue?.includes(dataItem[fieldValue]) ? 'checked' : '',
          ]"
        >
          <td
            class="cbb__item__col"
            v-for="(item, index) in infor"
            :key="index"
          >
            {{ dataItem[item.name] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "base-combobox-table",
  props: [
    "width",
    "minHeight",
    "data",
    "infor",
    "fieldDisplay",
    "fieldValue",
    "tabindex",
    "modelValue",
    "isDisable",
    "test"
  ],
  data() {
    return {
      isShowList: false,
      dropDownX: null,
      dropDownY: null,
      inputWidth: 0,
      inputHeight: 0,
      textSearch: "",
      dataChanged: [],
      listOptionSelected: [],
      fieldDispl: this.infor?.find((item) => {
        return item.isInputDisplay;
      }).name,
    };
  },

  computed: {
    //X??? l?? tag s??? hi???n th???
    handleDisplayOptionSelected() {
      return (id) => {
        const res = this.data.find((item) => {
          return item[this.fieldValue] == id;
        });
        if (res) {
          return res[this.fieldDispl];
        }
        return null;
      };
    },
    minHeightHandled() {
      return this.minHeight - 2;
    },
  },

  watch: {
    //Add s??? ki???n click outside khi list ??ang ???????c show
    isShowList(value) {
      if (value) {
        document.addEventListener("click", this.focusOutSide);
      } else {
        document.removeEventListener("click", this.focusOutSide);
      }
    },
    modelValue(newValue) {
      this.listOptionSelected = newValue || [];
    },
    test(newValue) {
      this.listOptionSelected = newValue || [];
    },
    // eslint-disable-next-line no-unused-vars
    listOptionSelected(value) {
      setTimeout(() => {
        const coordinates = this.$refs.input_wrap.getBoundingClientRect();
        const inputWrapHeight = this.$refs.input_wrap.offsetHeight;
        if (this.dropDownY) this.dropDownY = coordinates.y + inputWrapHeight;
      }, 0);
    },
    dropDownY(value) {
      if (value == 0) {
        this.dropDownY = null;
      }
    },
  },

  methods: {
    //X??? l?? khi click ra ngo??i combobox => ???n ??i
    focusOutSide(e) {
      if (
        !e.target.classList.contains("cbb__icon-wrap") &&
        !e.target.classList.contains("cbb__icon-hide") &&
        !e.target.classList.contains("cbb__icon-show") &&
        !e.target.classList.contains("cbb__item__col")
      ) {
        this.isShowList = false;
      }
    },
    //T??nh to??n v??? tr?? list option khi hi???n th???
    setPositionList() {
      setTimeout(() => {
        const coordinates = this.$refs.input_wrap.getBoundingClientRect();
        this.inputHeight = this.$refs.input_wrap.offsetHeight;
        this.inputWidth = this.$refs.input_wrap.offsetWidth;
        this.dropDownX = coordinates.x;
        this.dropDownY = coordinates.y + this.inputHeight;
      }, 0);
    },
    //x??? l?? khi click v??o icon ???n hi???n
    handleClickDisplayIcon() {
      if (!this.isDisable) {
        if (!this.isShowList) {
          this.setPositionList();
          this.dataChanged = this.data;
        }
        this.isShowList = !this.isShowList;
      }
    },
    handleClickOption(dataItem) {
      this.textSearch = "";
      //Add th??m option v??o list n???u ch??a c?? trong list, else => xo?? ra kh???i list
      const valueSelected = dataItem[this.fieldValue];
      if (!this.listOptionSelected.includes(valueSelected)) {
        this.listOptionSelected = [
          ...this.listOptionSelected,
          dataItem[this.fieldValue],
        ];
        // this.$emit("update:modelValue", [...this.modelValue, valueSelected]);
        this.$emit("update:modelValue", this.listOptionSelected);
      } else {
        this.listOptionSelected = this.listOptionSelected.filter((item) => {
          return item != dataItem[this.fieldValue];
        });
        this.$emit("update:modelValue", this.listOptionSelected);
      }
    },
    //X??? l?? khi input thay ?????i
    handleOnInput() {
      this.isShowList = true;
      //N???u input null => show to??n b??? data
      if (!this.textSearch) {
        this.dataChanged = this.data;
        //else => show nh???ng data ph?? h???p v???i value input
      } else {
        this.dataChanged = this.data.filter((item) => {
          let isMap = null;
          isMap = this.infor.find((inforItem) => {
            return item[inforItem.name]
              .toUpperCase()
              .includes(this.textSearch.toUpperCase());
          });
          if (isMap) {
            return true;
          } else {
            return false;
          }
        });
      }
    },
    handleClickAdd()
    {
      this.$emit("handleClickAdd")
    },

    //Xo?? ??i option ??ang ???????c ch???n v?? emit l???i value
    handleClickRemoveTag(id) {
      this.listOptionSelected = this.listOptionSelected.filter((item) => {
        return item != id;
      });
      this.$emit("update:modelValue", this.listOptionSelected);
    },

    //X??? l?? key down input
    handleKeyDownInput(e) {
      const allElementShowing = this.$refs.listCbb.querySelectorAll("tr");
      switch (e.keyCode) {
        //arrow down
        case 40:
          //Khi ??ang show list item
          if (this.isShowList) {
            if (allElementShowing.length > 0) {
              const currElement = this.$refs.listCbb.querySelector("tr.active");
              if (!currElement) {
                this.$refs.listCbb
                  .querySelector("tr.multiSelectItem-0")
                  .classList.add("active");
              } else if (
                currElement.classList.contains(
                  `multiSelectItem-${allElementShowing.length - 1}`
                )
              ) {
                currElement.classList.remove("active");
                this.$refs.listCbb
                  .querySelector("tr.multiSelectItem-0")
                  .classList.add("active");
              } else {
                currElement.classList.remove("active");
                currElement.nextElementSibling.classList.add("active");
              }
            }
          }
          break;
        case 38:
          //Khi ??ang show list item
          if (this.isShowList) {
            if (allElementShowing.length > 0) {
              const currElement = this.$refs.listCbb.querySelector("tr.active");
              if (!currElement) {
                this.$refs.listCbb
                  .querySelector(
                    `tr.multiSelectItem-${allElementShowing.length - 1}`
                  )
                  .classList.add("active");
              } else if (currElement.classList.contains(`multiSelectItem-0`)) {
                currElement.classList.remove("active");
                this.$refs.listCbb
                  .querySelector(
                    `tr.multiSelectItem-${allElementShowing.length - 1}`
                  )
                  .classList.add("active");
              } else {
                currElement.classList.remove("active");
                currElement.previousElementSibling.classList.add("active");
              }
            }
          }
          break;
        //enter
        case 13:
          //m??? list option n???u ??ang ????ng, else => ????ng n???u ??ang m???
          if (!this.isShowList) {
            //X??t v??? tr?? hi???n th???
            this.setPositionList();
            this.isShowList = true;
            this.dataChanged = this.data;
          } else {
            //N???u ??ang m???
            //G???i s??? ki???n click v??o option
            const currElement = this.$refs.listCbb.querySelector("tr.active");
            if (currElement) {
              currElement.click();
            }
            this.isShowList = false;
          }

          break;
        case 8:
          if (!this.textSearch) {
            this.listOptionSelected.pop();
            this.setPositionList();
            //this.$emit("update:modelValue", this.listOptionSelected);
          }
          break;
        default:
          break;
      }
    },
  },
};
</script>

<style  lang="scss">

@import "../../css/common/BaseMultiSelectTB.scss";
</style>
