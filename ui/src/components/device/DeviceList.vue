<template>
  <fragment>
    <v-card-text class="pa-0">
      <v-data-table
        class="elevation-1"
        :headers="headers"
        :items="getListDevices"
        item-key="uid"
        :sort-by="['started_at']"
        :sort-desc="[true]"
        :items-per-page="10"
        :footer-props="{'items-per-page-options': [10, 25, 50, 100]}"
        :server-items-length="getNumberDevices"
        :options.sync="pagination"
        :disable-sort="true"
        :search="search"
      >
        <template v-slot:item.online="{ item }">
          <v-icon
            v-if="item.online"
            color="success"
          >
            check_circle
          </v-icon>
          <v-tooltip
            v-else
            bottom
          >
            <template #activator="{ on }">
              <v-icon v-on="on">
                check_circle
              </v-icon>
            </template>
            <span>last seen {{ item.last_seen | moment("from", "now") }}</span>
          </v-tooltip>
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

        <template v-slot:item.namespace="{ item }">
          <v-chip class="list-itens">
            {{ address(item) }}<v-icon
              v-clipboard="() => address(item)"
              v-clipboard:success="showCopySnack"
              small
              right
              @click.stop
            >
              mdi-content-copy
            </v-icon>
          </v-chip>
        </template>

        <template v-slot:item.actions="{ item }">
          <v-tooltip bottom>
            <template #activator="{ on }">
              <v-icon
                class="icons"
                v-on="on"
                @click="detailsDevice(item)"
              >
                info
              </v-icon>
            </template>
            <span>Details</span>
          </v-tooltip>

          <TerminalDialog
            v-if="item.online"
            :uid="item.uid"
          />

          <DeviceDelete
            :uid="item.uid"
            @update="refresh"
          />
        </template>
      </v-data-table>
    </v-card-text>
    <v-snackbar
      v-model="copySnack"
      :timeout="3000"
    >
      Device SSHID copied to clipboard
    </v-snackbar>
  </fragment>
</template>

<script>

import TerminalDialog from '@/components/terminal/TerminalDialog';
import DeviceIcon from '@/components/device//DeviceIcon';
import DeviceDelete from '@/components/device//DeviceDelete';

export default {
  name: 'DeviceList',

  components: {
    TerminalDialog,
    DeviceIcon,
    DeviceDelete,
  },

  data() {
    return {
      hostname: window.location.hostname,
      dialogDelete: false,
      pagination: {},
      copySnack: false,
      editName: '',
      search: '',
      headers: [
        {
          text: 'Online',
          value: 'online',
          align: 'center',
        },
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
          text: 'SSHID',
          value: 'namespace',
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
    getListDevices() {
      return this.$store.getters['devices/list'];
    },

    getNumberDevices() {
      return this.$store.getters['devices/getNumberDevices'];
    },
  },

  watch: {
    pagination: {
      handler() {
        this.getDevices();
      },
      deep: true,
    },

    search() {
      this.getDevices();
    },
  },

  methods: {
    async getDevices() {
      let encodedFilter = null;

      if (this.search) {
        const filter = [{ type: 'property', params: { name: 'name', operator: 'like', value: this.search } }];
        encodedFilter = btoa(JSON.stringify(filter));
      }

      const data = {
        perPage: this.pagination.itemsPerPage,
        page: this.pagination.page,
        filter: encodedFilter,
        status: 'accepted',
      };

      this.$store.dispatch('devices/fetch', data);
    },

    detailsDevice(value) {
      this.$router.push(`/device/${value.uid}`);
    },

    address(item) {
      return `${item.namespace}.${item.name}@${this.hostname}`;
    },

    copy(device) {
      this.$clipboard(device.uid);
    },

    showCopySnack() {
      this.copySnack = true;
    },

    refresh() {
      this.getDevices();
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
