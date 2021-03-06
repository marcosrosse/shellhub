<template>
  <fragment>
    <v-card-text class="pa-0">
      <v-data-table
        class="elevation-1"
        :headers="headers"
        :items="getListRejectedDevices"
        item-key="uid"
        :sort-by="['started_at']"
        :sort-desc="[true]"
        :items-per-page="10"
        :footer-props="{'items-per-page-options': [10, 25, 50, 100]}"
        :server-items-length="getNumberRejectedDevices"
        :options.sync="pagination"
        :disable-sort="true"
        :search="search"
      >
        <template slot="no-data">
          There are no more pending devices
        </template>

        <template v-slot:item.hostname="{ item }">
          <router-link :to="{ name: 'detailsDevice', params: { id: item.uid } }">
            {{ item.name }}
          </router-link>
        </template>

        <template v-slot:item.info.pretty_name="{ item }">
          <DeviceIcon :icon-name="item.info.id" />
          {{ item.info.pretty_name }}
        </template>

        <template v-slot:item.request_time="{ item }">
          {{ item.last_seen | moment("ddd, MMM Do YY, h:mm:ss a") }}
        </template>

        <template v-slot:item.actions="{ item }">
          <DeviceActionButton
            :uid="item.uid"
            action="accept"
            @update="refresh"
          />

          <DeviceActionButton
            :uid="item.uid"
            action="remove"
            @update="refresh"
          />
        </template>
      </v-data-table>
    </v-card-text>
  </fragment>
</template>

<script>

import DeviceIcon from '@/components/device//DeviceIcon';
import DeviceActionButton from '@/components/device/DeviceActionButton';

export default {
  name: 'DeviceList',

  components: {
    DeviceIcon,
    DeviceActionButton,
  },

  data() {
    return {
      pagination: {},
      copySnack: false,
      editName: '',
      search: '',
      headers: [
        {
          text: 'Hostname',
          value: 'hostname',
          align: 'center',
        },
        {
          text: 'Operating System',
          value: 'info.pretty_name',
          align: 'center',
        },
        {
          text: 'Request Time',
          value: 'request_time',
          align: 'center',
        },
        {
          text: 'Actions',
          value: 'actions',
          align: 'center',
          sortable: false,
        },
      ],
    };
  },

  computed: {
    getListRejectedDevices() {
      return this.$store.getters['devices/list'];
    },

    getNumberRejectedDevices() {
      return this.$store.getters['devices/getNumberDevices'];
    },
  },

  watch: {
    pagination: {
      handler() {
        this.getRejectedDevices();
      },
      deep: true,
    },

    search() {
      this.getRejectedDevices();
    },
  },

  methods: {
    async getRejectedDevices() {
      let filter = null;
      let encodedFilter = null;

      if (this.search) {
        filter = [{ type: 'property', params: { name: 'name', operator: 'like', value: this.search } }];
        encodedFilter = btoa(JSON.stringify(filter));
      }

      const data = {
        perPage: this.pagination.itemsPerPage,
        page: this.pagination.page,
        filter: encodedFilter,
        status: 'rejected',
      };

      this.$store.dispatch('devices/fetch', data);
    },

    refresh() {
      this.getRejectedDevices();
    },
  },
};

</script>

<style scoped>

.list-itens {
  font-family: monospace;
}

.icons{
  margin-right: 4px;
}

</style>
