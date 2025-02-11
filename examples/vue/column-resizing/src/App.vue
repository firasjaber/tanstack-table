<script setup lang="ts">
import {
  FlexRender,
  getCoreRowModel,
  useVueTable,
  createColumnHelper,
  ColumnResizeMode,
} from '@tanstack/vue-table'
import { computed, ref, watch } from 'vue'

type Person = {
  firstName: string
  lastName: string
  age: number
  visits: number
  status: string
  progress: number
}

const defaultData: Person[] = [
  {
    firstName: 'tanner',
    lastName: 'linsley',
    age: 24,
    visits: 100,
    status: 'In Relationship',
    progress: 50,
  },
  {
    firstName: 'tandy',
    lastName: 'miller',
    age: 40,
    visits: 40,
    status: 'Single',
    progress: 80,
  },
  {
    firstName: 'joe',
    lastName: 'dirte',
    age: 45,
    visits: 20,
    status: 'Complicated',
    progress: 10,
  },
]

const columnHelper = createColumnHelper<Person>()

const columns = [
  columnHelper.group({
    header: 'Name',
    footer: props => props.column.id,
    columns: [
      columnHelper.accessor('firstName', {
        cell: info => info.getValue(),
        footer: props => props.column.id,
      }),
      columnHelper.accessor(row => row.lastName, {
        id: 'lastName',
        cell: info => info.getValue(),
        header: () => 'Last Name',
        footer: props => props.column.id,
      }),
    ],
  }),
  columnHelper.group({
    header: 'Info',
    footer: props => props.column.id,
    columns: [
      columnHelper.accessor('age', {
        header: () => 'Age',
        footer: props => props.column.id,
        size: 50,
        maxSize: 50,
      }),
      columnHelper.group({
        header: 'More Info',
        columns: [
          columnHelper.accessor('visits', {
            header: () => 'Visits',
            footer: props => props.column.id,
            size: 70,
          }),
          columnHelper.accessor('status', {
            header: 'Status',
            footer: props => props.column.id,
          }),
          columnHelper.accessor('progress', {
            header: 'Profile Progress',
            footer: props => props.column.id,
          }),
        ],
      }),
    ],
  }),
]

const data = ref(defaultData)

const columnResizeMode = ref<ColumnResizeMode>('onChange')

const rerender = () => {
  data.value = defaultData
}

const table = computed(() =>
  useVueTable({
    get data() {
      return data.value
    },
    columns,
    columnResizeMode: columnResizeMode.value,
    getCoreRowModel: getCoreRowModel(),
  })
)
</script>

<template>
  <div class="p-2">
    <div class="p-2">
      <label class="mr-4"
        >Resizing mode:
        <select v-model="columnResizeMode">
          <option>onChange</option>
          <option>onEnd</option>
        </select>
      </label>
    </div>
    <table>
      <thead>
        <tr
          v-for="headerGroup in table.getHeaderGroups()"
          :key="headerGroup.id"
        >
          <th
            v-for="header in headerGroup.headers"
            :key="header.id"
            :colSpan="header.colSpan"
            :style="{
              width: header.column.getSize() + 'px',
            }"
          >
            <FlexRender
              v-if="!header.isPlaceholder"
              :render="header.column.columnDef.header"
              :props="header.getContext()"
            />
            <div
              class="resizer"
              @mousedown="
                (event: MouseEvent) => header.getResizeHandler()(event)
              "
              @touchstart="
                (event: TouchEvent) => header.getResizeHandler()(event)
              "
              :class="{ isResizing: header.column.getIsResizing() }"
              :key="header.id + 'resizer'"
            />
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in table.getRowModel().rows" :key="row.id">
          <td v-for="cell in row.getVisibleCells()" :key="cell.id">
            <FlexRender
              :render="cell.column.columnDef.cell"
              :props="cell.getContext()"
            />
          </td>
        </tr>
      </tbody>
      <tfoot>
        <tr
          v-for="footerGroup in table.getFooterGroups()"
          :key="footerGroup.id"
        >
          <th
            v-for="header in footerGroup.headers"
            :key="header.id"
            :colSpan="header.colSpan"
          >
            <FlexRender
              v-if="!header.isPlaceholder"
              :render="header.column.columnDef.footer"
              :props="header.getContext()"
            />
          </th>
        </tr>
      </tfoot>
    </table>
    <div class="h-4" />
    <button @click="rerender" class="border p-2">Rerender</button>
    <pre>
      {{
        '\n' +
        JSON.stringify(
          {
            columnSizing: table.getState().columnSizing,
            columnSizingInfo: table.getState().columnSizingInfo,
          },
          null,
          2
        )
      }}
    </pre>
  </div>
</template>

<style>
html {
  font-family: sans-serif;
  font-size: 14px;
}

table {
  border: 1px solid lightgray;
}

tbody {
  border-bottom: 1px solid lightgray;
}

th {
  border-bottom: 1px solid lightgray;
  border-right: 1px solid lightgray;
  padding: 2px 4px;
  position: relative;
}

td {
  border-bottom: 1px solid lightgray;
  border-right: 1px solid lightgray;
}

tfoot {
  color: gray;
}

tfoot th {
  font-weight: normal;
}

.resizer {
  position: absolute;
  right: 0;
  top: 0;
  height: 100%;
  width: 5px;
  background: #cbd5e1;
  cursor: col-resize;
  user-select: none;
  touch-action: none;
}

.resizer.isResizing {
  background: #94a3b8;
  opacity: 1;
}

@media (hover: hover) {
  .resizer {
    opacity: 0;
  }

  *:hover > .resizer {
    opacity: 1;
  }
}
</style>
