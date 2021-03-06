<template>
  <div>
    <table class="data-table">
      <tr>
        <th
          v-for="(header, headerIndex) in headers"
          :key="headerIndex"
          @click="toggleSortBy(header)"
        >
           <span class="header">{{ convertAttributeToLabel(header) }}</span>
          <div
            v-if="sortBy.attribute === header"
            class="carat-icon"
            :class="[
              { sortByDescending: sortBy.order === 'descending' },
              { sortByAscending: sortBy.order === 'ascending' },
            ]"
          >&lt;</div>
        </th>
        <th v-if="actions && actions.length">Actions</th>
      </tr>
      <tr v-for="(row, rowIndex) in paginatedData" :key="rowIndex">
        <td
          v-for="(attribute, attributeIndex) in headers"
          :key="attributeIndex"
          :class="{ action: attribute === 'action' }"
        >
          {{ row[attribute] }}
        </td>
        <td v-if="actions && actions.length">
          <p v-for="(action, actionIndex) in actions" :key="actionIndex" @click="handleClick(action, row)" class="action">{{convertAttributeToLabel(action)}}</p>
        </td>
      </tr>
    </table>
    <div v-if="perPage && numPages > 1" class="pagination-container">
      <div class="pagination">
        <span @click="page -= 1" class="prev pagination-icon" :class="{ disabled: page === 1 }">&lt;</span>
        <span
          v-for="number in minifiedPaginatorOptions"
          :key="number"
          class="pagination-number"
          :class="{ active: page === number + 1 }"
          @click="setPageNumber(number)"
        >
          <template v-if="number === 'left-ellipsis' || number === 'right-ellipsis'">... </template
          ><template v-else>{{ number + 1 }} </template></span
        >
        <span
          class="pagination-icon"
          @click="page += 1"
          :class="{ disabled: page === minifiedPaginatorOptions[minifiedPaginatorOptions.length - 1] + 1 }"
        >&gt;</span>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'Data-Table',
  props: {
    data: {
      type: Array,
      required: true,
    },
    actions: {
      type: Array,
      required: false,
    },
    perPage: {
      type: Number,
      required: false,
    },
  },
  data() {
    return {
      sortBy: {
        attribute: '',
        order: 'ascending',
      },
      page: 1,
    };
  },
  computed: {
    headers() {
      const headers = [];
      this.data.forEach((row) => {
        const rowKeys = Object.keys(row);
        rowKeys.forEach((key) => {
          if (!headers.includes(key)) headers.push(key);
        });
      });
      return headers;
    },
    sortedData() {
      // todo: sorting not accurate
      const { data } = this;
      if (this.sortBy.attribute && this.sortBy.attribute.length) {
        if (this.sortBy.order === 'descending') {
          data.sort((a, b) => (a[this.sortBy.attribute] > b[this.sortBy.attribute] ? 1 : -1));
        }
        if (this.sortBy.order === 'ascending') {
          data.sort((a, b) => (b[this.sortBy.attribute] < a[this.sortBy.attribute] ? 1 : -1));
        }
      }
      return data;
    },
    paginatedData() {
      if (this.perPage && this.numPages && this.page) {
        return this.sortedData.slice(this.perPage * (this.page - 1), this.perPage * this.page);
      }
      return this.sortedData;
    },
    numPages() {
      if (this.perPage && this.data && Math.ceil(this.data.length / this.perPage) > 1) {
        return Math.ceil(this.data.length / this.perPage);
      }
      return null;
    },
    paginatorOptions() {
      const paginatorOptions = [];
      for (let i = 0; i < this.numPages; i += 1) {
        paginatorOptions.push(i);
      }
      return paginatorOptions;
    },
    minifiedPaginatorOptions() {
      if (this.paginatorOptions.length < 6) {
        return this.paginatorOptions;
      }
      if (this.page < 6) {
        const minifiedPaginatorOptions = this.paginatorOptions.slice(0, 5);
        minifiedPaginatorOptions.push('right-ellipsis');
        minifiedPaginatorOptions.push(this.paginatorOptions.length - 1);
        return minifiedPaginatorOptions;
      }
      if (this.page > this.paginatorOptions.length - 5) {
        const minifiedPaginatorOptions = this.paginatorOptions.slice(
          this.page - (5 - (this.paginatorOptions.length - this.page)),
          this.paginatorOptions.length,
        );
        minifiedPaginatorOptions.unshift('left-ellipsis');
        minifiedPaginatorOptions.unshift(0);
        return minifiedPaginatorOptions;
      }
      const minifiedPaginatorOptions = this.paginatorOptions.slice(this.page - 3, this.page + 2);
      minifiedPaginatorOptions.unshift('left-ellipsis');
      minifiedPaginatorOptions.unshift(0);
      minifiedPaginatorOptions.push('right-ellipsis');
      minifiedPaginatorOptions.push(this.paginatorOptions.length - 1);
      return minifiedPaginatorOptions;
    },
  },
  methods: {
    convertAttributeToLabel(attribute) {
      const result = attribute.replace(/([A-Z])/g, ' $1');
      const finalResult = result.charAt(0).toUpperCase() + result.slice(1);
      return finalResult;
    },
    toggleSortBy(attributeName) {
      if (this.sortBy.attribute === attributeName) {
        if (this.sortBy.order === 'ascending') {
          this.sortBy.order = 'descending';
        } else {
          this.sortBy.attribute = '';
          this.sortBy.order = 'ascending';
        }
      } else {
        this.sortBy.attribute = attributeName;
        this.sortBy.order = 'ascending';
      }
    },
    setPageNumber(number) {
      if (number === 'left-ellipsis') {
        this.page = this.minifiedPaginatorOptions[2];
      } else if (number === 'right-ellipsis') {
        this.page = this.minifiedPaginatorOptions[this.minifiedPaginatorOptions.length - 3] + 2;
      } else {
        this.page = number + 1;
      }
    },
    handleClick(action, row) {
      this.$emit(action, row);
    },
  },
  watch: {
    numPages() {
      if (this.numPages !== null) {
        this.page = 1;
      }
    },
  },
};
</script>

<style scoped>
.data-table {
  width: 100%;
  margin-bottom: 40px;
  border-collapse: collapse;
}
th {
  padding: 15px;
  font-size: 1.2rem;
  font-weight: bold;
  color: white;
  text-align: left;
  cursor: pointer;
  background: black;
  border: 1px solid black;
}
.header {
  padding-right: 10px;
}
.carat-icon {
  width: 16px;
  display: inline-block;
  transform: rotate(90deg);
  transition: all 0.3s ease-in-out;
}
.sortByAscending {
  transition: all 0.3s ease-in-out;
  transform: rotate(-90deg);
}
td {
  padding: 15px;
  font-size: 1.2rem;
  text-align: left;
  border: 1px solid lightgray;
}
.action {
  font-weight: bold;
  cursor: pointer;
}
.pagination-container {
  display: flex;
  align-items: center;
  justify-content: center;
}
.pagination {
  display: flex;
  align-items: center;
  height: 20px;
  font-size: 1.2rem;
  text-align: center;
}
.pagination-number {
  padding: 0 25px;
   cursor: pointer;
}
.active {
  font-weight: bold;
}
.pagination-icon {
  height: 100%;
  padding: 0 35px;
  cursor: pointer;
}
.disabled {
  pointer-events: none;
  opacity: 0.5;
}
</style>
