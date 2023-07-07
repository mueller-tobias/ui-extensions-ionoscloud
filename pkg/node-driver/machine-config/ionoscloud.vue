<script>
import Loading from '@shell/components/Loading';
import { Banner } from '@components/Banner';
import CreateEditView from '@shell/mixins/create-edit-view';
import LabeledSelect from '@shell/components/form/LabeledSelect';
import { LabeledInput } from '@components/Form/LabeledInput';
import { stringify } from '@shell/utils/error';
import { _VIEW } from '@shell/config/query-params';
import FileSelector from '../components/FileSelector';


function initLocation() {
  return {
    options:  [
      {
        label: 'Las Vegas, USA',
        value: {'value': 'us/las', 'name': 'Las Vegas, USA'}
      },
      {
        label:'Newark, USA',
        value: {'value': 'us/ewr', 'name':'Newark, USA'}
      },
      {
        label: 'Frankfurt, Germany',
        value: {'value': 'de/fra', 'name': 'Frankfurt, Germany'}
      },
      {
        label: 'Berlin, Germany',
        value: {'value': 'de/txl', 'name': 'Berlin, Germany'}
      },
      {
        label: 'London, UK',
        value: {'value': 'gb/lhr', 'name': 'London, UK'}
      },
      {
        label: 'Logroño, Spain',
        value: {'value': 'es/vit', 'name': 'Logroño, Spain'}
      },
      {
        label: 'Paris, France',
        value: {'value': 'fr/par', 'name': 'Paris, France'}
      },
    ],
    selected: {'value': 'us/las', 'name': 'Las Vegas, USA'},
    busy:     false,
    enabled:  false,
  };
}

function initServerType() {
  return {
    options:  [
      {
        label: 'Enterprise',
        value: {'value': 'ENTERPRISE', 'name': 'Enterprise'}
      },
      {
        label:'Cube',
        value: {'value': 'CUBE', 'name':'Cube'}
      },
    ],
    selected: {'value': 'ENTERPRISE', 'name': 'Enterprise'},
    busy:     false,
    enabled:  false,
  };
}

function initServerZone() {
  return {
    options:  [
      {
        label: 'AUTO',
        value: {'value': 'AUTO', 'name': 'AUTO'}
      },
      {
        label:'ZONE_1',
        value: {'value': 'ZONE_1', 'name':'ZONE_1'}
      },
      {
        label:'ZONE_2',
        value: {'value': 'ZONE_2', 'name':'ZONE_2'}
      },
    ],
    selected: {'value': 'AUTO', 'name': 'AUTO'},
    busy:     false,
    enabled:  false,
  };
}

function initVolumeZone() {
  return {
    options:  [
      {
        label: 'AUTO',
        value: {'value': 'AUTO', 'name': 'AUTO'}
      },
      {
        label:'ZONE_1',
        value: {'value': 'ZONE_1', 'name':'ZONE_1'}
      },
      {
        label:'ZONE_2',
        value: {'value': 'ZONE_2', 'name':'ZONE_2'}
      },
      {
        label:'ZONE_3',
        value: {'value': 'ZONE_3', 'name':'ZONE_3'}
      },
    ],
    selected: {'value': 'AUTO', 'name': 'AUTO'},
    busy:     false,
    enabled:  false,
  };
}

export default {
  components: {
    Banner, FileSelector, Loading, LabeledInput, LabeledSelect
  },

  mixins: [CreateEditView],

  props: {
    uuid: {
      type:     String,
      required: true,
    },

    cluster: {
      type:    Object,
      default: () => ({})
    },

    credentialId: {
      type:     String,
      required: true,
    },

    disabled: {
      type:    Boolean,
      default: false
    },

    busy: {
      type:    Boolean,
      default: false
    },

    provider: {
      type:     String,
      required: true,
    }
  },

  async fetch() {
    this.errors = [];
    if ( !this.credentialId ) {
      return;
    }

    if (this.mode === _VIEW) {
      this.initForViewMode();

      return;
    }

    this.$emit('validationChanged', true);
  },

  data() {
    return {
      authenticating:      false,
      ready:               false,
      os:                  null,
      password:            null,
      havePassword:        false,
      location:            initLocation(),
      serverType:          initServerType(),
      serverZone:          initServerZone(),
      volumeZone:          initVolumeZone(),
      sshUser:             this.value?.sshUser || 'root',
      privateKeyFile:      this.value?.privateKeyFile || '',
      filename:            this.value?.privateKeyFile ? 'Private Key Provided' : '',
      privateKeyFieldType: 'password',
      errors:              null,
    };
  },

  watch: {
    'credentialId'() {
      this.$fetch();
    },
  },

  methods: {
    stringify,

    initForViewMode() {
      this.fakeSelectOptions(this.Location, this.value?.location);
      this.fakeSelectOptions(this.ServerType, this.value?.serverType);
      this.fakeSelectOptions(this.ServerZone, this.value?.serverZone);
      this.fakeSelectOptions(this.VolumeZone, this.value?.volumeZone);
    },

    fakeSelectOptions(list, value) {
      list.busy = false;
      list.enabled = false;
      list.options = [];

      if (value) {
        list.options.push({
          label: value,
          value,
        });
      }

      list.selected = value;
    },

    onPrivateKeyFileSelected(v) {
      this.filename = v.file.name;
      this.privateKeyFile = v.data;

      // On initial load, filename is shown as a password as we don't know what the filename was that was used - we just want to indicate there is a vlue
      // When a file is chosen, change the type to text, so that the user can see the filename of the file that they chose
      this.privateKeyFieldType = 'text';

      this.$emit('validationChanged', true);
    },

    syncValue() {
      // Note: We don't need to provide password as this is picked up via the credential

      // Copy the values from the form to the correct places on the value
      this.value.location = this.location.selected?.name;
      this.value.serverType = this.serverType.selected?.name;
      this.value.serverZone = this.serverZone.selected?.name;
      this.value.volumeZone = this.volumeZone.selected?.name;
    },

    test() {
      this.syncValue();
    }
  }
};
</script>

<template>
  <div>
    <Loading
      v-if="$fetchState.pending"
      :delayed="true"
    />
    <div v-if="errors.length">
      <div
        v-for="(err, idx) in errors"
        :key="idx"
      >
        <Banner
          color="error"
          :label="stringify(err)"
        />
      </div>
    </div>
    <div>
      <div class="ionoscloud-config">
        <div class="title">
          ionoscloud Configuration
        </div>
      </div>
      <div class="row mt-10">
        <div class="col span-3">
          <LabeledSelect
            v-model="location.selected"
            label="Location"
            :options="location.options"
            :loading="location.busy"
            :searchable="false"
          />
        </div>
        <div class="col span-3">
          <LabeledSelect
            v-model="serverType.selected"
            label="ServerType"
            :options="serverType.options"
            :loading="serverType.busy"
            :searchable="false"
          />
        </div>
        <div class="col span-3">
          <LabeledSelect
            v-model="serverZone.selected"
            label="ServerZone"
            :options="serverZone.options"
            :loading="serverZone.busy"
            :searchable="false"
          />
        </div>
        <div class="col span-3">
          <LabeledSelect
            v-model="volumeZone.selected"
            label="VolumeZone"
            :options="volumeZone.options"
            :loading="volumeZone.busy"
            :searchable="false"
          />
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
  .file-button {
    align-items: center;
    position: absolute;
    top: 0;
    right: 0;
    height: 100%;
    display: flex;

    > .file-selector {
      height: calc($input-height - 2px);
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
    }
  }

  .ionoscloud-config {
    display: flex;
    align-items: center;

    > .title {
      font-weight: bold;
      padding: 4px 0;
    }

    > .loading {
      margin-left: 20px;
      display: flex;
      align-items: center;

      > i {
        margin-right: 4px;;
      }
    }
  }
</style>
