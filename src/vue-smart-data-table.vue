<template>
  <div>
    <table class="data-table">
      <tr>
        <th
          v-for="(header, headerIndex) in headers"
          :key="headerIndex"
          @click="toggleSortBy(header)"
          :class="[
            { sortByDescending: sortBy.attribute === header && sortBy.order === 'descending' },
            { sortByAscending: sortBy.attribute === header && sortBy.order === 'ascending' },
          ]"
        >
          <span>{{ convertAttributeToLabel(header) }}</span>
          <CaratIcon
            v-if="sortBy.attribute === header"
            :class="[
              { sortByDescending: sortBy.order === 'descending' },
              { sortByAscending: sortBy.order === 'ascending' },
            ]"
          />
        </th>
        <th v-if="actions && actions.length">Actions</th>
      </tr>
      <tr v-for="(row, rowIndex) in paginatedData" :key="rowIndex">
        <td v-for="(attribute, attributeIndex) in headers" :key="attributeIndex">
          {{ row[attribute] }}
        </td>
        <td v-if="actions && actions.length">
          <ul>
            <li v-for="(action, actionIndex) in actions" :key="actionIndex" @click="handleClick(row, action)">
              {{ action }}
            </li>
          </ul>
        </td>
      </tr>
    </table>
    <div v-if="perPage && numPages > 1" class="pagination-container">
      <div class="pagination">
        <NextArrow @click="page -= 1" class="prev" :class="{ disabled: page === 1 }" />
        <span
          v-for="number in minifiedPaginatorOptions"
          :key="number"
          :class="{ active: page === number + 1 }"
          @click="setPageNumber(number)"
        >
          <template v-if="number === 'left-ellipsis' || number === 'right-ellipsis'">... </template
          ><template v-else>{{ number + 1 }} </template></span
        >
        <NextArrow
          @click="page += 1"
          :class="{ disabled: page === minifiedPaginatorOptions[minifiedPaginatorOptions.length - 1] + 1 }"
        />
      </div>
    </div>
  </div>
</template>

<script>
// import PrevArrow from '../assets/icons/previous_arrow.svg';
import NextArrow from '../assets/icons/next_arrow.svg';
import CaratIcon from '../assets/icons/down_carat.svg';

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
  components: {
    // PrevArrow,
    NextArrow,
    CaratIcon,
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
      const { data } = this;
      if (this.sortBy.attribute && this.sortBy.attribute.length) {
        if (this.sortBy.order === 'descending') {
          data.sort((a, b) => (a[this.sortBy.attribute] > b[this.sortBy.attribute] ? 1 : -1));
        }
        if (this.sortBy.order === 'ascending') {
          data.sort((a, b) => (b[this.sortBy.attribute] > a[this.sortBy.attribute] ? 1 : -1));
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
      // const currentPage = this.page;
      if (number === 'left-ellipsis') {
        // this.page -= 6 - (this.paginatorOptions.length - currentPage);
        this.page = this.minifiedPaginatorOptions[2];
      } else if (number === 'right-ellipsis') {
        // this.page += 6 - currentPage;
        this.page = this.minifiedPaginatorOptions[this.minifiedPaginatorOptions.length - 3] + 2;
      } else {
        this.page = number + 1;
      }
    },
    handleClick(row, action) {
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

<style lang="scss" scoped>
.data-table {
  width: 100%;
  margin-bottom: 40px;
  border-collapse: collapse;
  tr {
    th {
      padding: 15px;
      font-size: 1.2rem;
      font-weight: bold;
      color: $kia-polar-white;
      text-align: left;
      cursor: pointer;
      background: $kia-midnight-black;
      border: 1px solid $kia-midnight-black;
      span {
        padding-right: 10px;
      }
      svg {
        width: 16px;
        transition: all 0.3 ease-in-out;
        &.sortByAscending {
          transition: all 0.3 ease-in-out;
          transform: rotate(180deg);
        }
      }
    }
    td {
      padding: 15px;
      font-size: 1.2rem;
      text-align: left;
      border: 1px solid $kia-light-gray;
      &.action {
        font-weight: bold;
        cursor: pointer;
      }
      ul {
        padding: 0;
        margin: 0;
        list-style: none;
        li {
          margin: 10px 0;
          font-weight: bold;
          cursor: pointer;
        }
      }
    }
  }
}
.pagination-container {
  display: flex;
  align-items: center;
  justify-content: center;
  .pagination {
    display: flex;
    align-items: center;
    height: 20px;
    font-size: 1.2rem;
    text-align: center;
    span {
      padding: 0 25px;
      cursor: pointer;
      &.active {
        font-weight: bold;
      }
    }
    svg {
      height: 100%;
      padding: 0 35px;
      cursor: pointer;
      &.prev {
        transform: rotate(180deg);
      }
      &.disabled {
        pointer-events: none;
        opacity: 0.5;
      }
    }
  }
}
</style>
