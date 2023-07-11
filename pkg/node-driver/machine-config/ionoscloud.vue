<script>
import Loading from '@shell/components/Loading';
import { Banner } from '@components/Banner';
import CreateEditView from '@shell/mixins/create-edit-view';
import LabeledSelect from '@shell/components/form/LabeledSelect';
import { LabeledInput } from '@components/Form/LabeledInput';
import { Checkbox } from '@components/Form/Checkbox';
import { StringList } from '@components/StringList';
import { TextArea } from '@components/Form/TextArea';
import { stringify } from '@shell/utils/error';
import { _VIEW } from '@shell/config/query-params';
import FileSelector from '../components/FileSelector';

import { LabeledTooltip } from '@rancher/components';

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

function initCpuFamily() {
  return {
    options:  [
      {
        label: 'Intel SKYLAKE (Europe)',
        value: {'value': 'INTEL_SKYLAKE', 'name': 'Intel SKYLAKE (Europe)'}
      },
      {
        label: 'AMD OPTERON (USA)',
        value: {'value': 'AMD_OPTERON', 'name': 'AMD OPTERON (USA)'}
      },
      {
        label: 'Intel XEON (USA)',
        value: {'value': 'INTEL_XEON', 'name': 'Intel XEON (USA)'}
      },
    ],
    selected: {'value': 'INTEL_SKYLAKE', 'name': 'Intel SKYLAKE (Europe)'},
    busy:     false,
    enabled:  false,
  };
}

function initDiskType() {
  return {
    options:  [
      {
        label: 'HDD',
        value: {'value': 'HDD', 'name': 'HDD'}
      },
      {
        label: 'SSD',
        value: {'value': 'SSD', 'name': 'SSD'}
      },
    ],
    selected: {'value': 'HDD', 'name': 'HDD'},
    busy:     false,
    enabled:  false,
  };
}

function validateIp(ip) {
  if (/^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(ip)) {
    return true;
  }
  return false;
}


export default {
  components: {
    Banner,
    FileSelector,
    Loading,
    Checkbox,
    StringList,
    LabeledInput,
    LabeledSelect,
    TextArea,
    LabeledTooltip
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
      cpuFamily:           initCpuFamily(),
      diskType:            initDiskType(),
      cores:               this.value?.cores || '2',
      ram:                 this.value?.ram || '2048',
      diskSize:            this.value?.diskSize || '50',
      image:               this.value?.image || 'ubuntu:20.04',
      imagePassword:       this.value?.imagePassword,
      userData:            this.value?.userData,
      sshUser:             this.value?.sshUser || 'root',
      sshInUserData:       this.value?.sshInUserData || false,
      datacenterId:        this.value?.datacenterId,
      datacenterName:      this.value?.datacenterName || 'docker-machine-data-center',
      lanId:               this.value?.lanId,
      lanName:             this.value?.lanName || 'docker-machine-lan',
      privateLan:          this.value?.privateLan || false,
      nicDhcp:             this.value?.nicDhcp || false,
      nicIps:              this.value?.nicIps || [],
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
      this.fakeSelectOptions(this.CpuFamily, this.value?.cpuFamily);
      this.fakeSelectOptions(this.DiskType, this.value?.diskType);
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

    onChangeNicIps(event) {
      for (let ip of event) {
        if (!validateIp(ip)) {
          alert("You have entered an invalid IP address!");
          return;
        }
      }

      this.nicIps = event;
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
      this.value.cpuFamily = this.cpuFamily.selected?.name;
      this.value.diskType = this.diskType.selected?.name;
      this.value.cores = this.cores;
      this.value.ram = this.ram;
      this.value.diskSize = this.diskSize;
      this.value.image = this.image;
      this.value.imagePassword = this.imagePassword;
      this.value.userData = this.userData;
      this.value.sshUser = this.sshUser;
      this.value.sshInUserData = this.sshInUserData;
      this.value.datacenterId = this.datacenterId;
      this.value.datacenterName = this.datacenterName;
      this.value.lanId = this.lanId;
      this.value.lanName = this.lanName;
      this.value.privateLan = this.privateLan;
      this.value.nicDhcp = this.nicDhcp;
      this.value.nicIps = this.nicIps;
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

      <div class="row mt-10">
        <div class="col span-3">
          <LabeledSelect
            v-model="cpuFamily.selected"
            label="CpuFamily"
            :options="cpuFamily.options"
            :loading="cpuFamily.busy"
            :searchable="false"
          />
        </div>
        <div class="col span-3">
          <LabeledInput
            v-model="cores"
            :mode="mode"
            :disabled="busy"
            :required="true"
            label="CPU Core Count"
          />
        </div>
        <div class="col span-6">
          <LabeledInput
            v-model="ram"
            :mode="mode"
            :disabled="busy"
            :required="true"
            label="RAM size (in MiB)"
          />
          <p class="help-block">Must be a multiple of 256</p>
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-6">
          <LabeledInput
            v-model="diskSize"
            :mode="mode"
            :disabled="busy"
            :required="true"
            label="Disk size (in GB)"
          />
        </div>
        <div class="col span-6">
          <LabeledSelect
            v-model="diskType.selected"
            label="DiskType"
            :options="diskType.options"
            :loading="diskType.busy"
            :searchable="false"
            :required="true"
          />
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-6">
          <LabeledInput
            v-model="image"
            :mode="mode"
            :disabled="busy"
            :required="true"
            label="Image Alias or ID"
          />
          <p class="help-block">You can use <a href="https://docs.ionos.com/cli-ionosctl" target="_blank" rel="noopener noreferrer">ionosctl image list [-F name=operatingSystem]</a></p>
        </div>
        <div class="col span-6">
          <LabeledInput
            v-model="imagePassword"
            :mode="mode"
            :disabled="busy"
            :required="true"
            label="Image Password"
            type="password"
          />
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-12">
          <label class="acc-label">Cloud init configuration.</label>
          <TextArea
            v-model="userData"
            :mode="mode"
            :disabled="busy"
            @input="userData=$event.target.value;"
          ></TextArea>
          <p class="help-block">Optional. <a href="https://cloudinit.readthedocs.io/en/latest/topics/examples.html" target="_blank" rel="noopener noreferrer">Cloud-init Documentation</a>.</p>
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-4">
          <LabeledInput
            v-model="sshUser"
            :mode="mode"
            :disabled="busy"
            label="SSH User"
          />
          <p class="help-block">Optional. User to connect to via SSH.</p>
        </div>
        <div class="col span-4">
          <Checkbox
            label="Send SSH in user data."
            v-model="sshInUserData"
            :mode="mode"
            :disabled="busy"
          />
          <p class="help-block">Should the driver only add the SSH info in the user data? (required for custom images).</p>
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-4">
          <LabeledInput
            v-model="datacenterId"
            :mode="mode"
            :disabled="busy"
            label="Datacenter ID"
          />
          <p class="help-block">Optional, UUID-4 format. If you want to use an existing IONOS Datacenter to host this VM, you can provide its ID here.</p>
        </div>
        <div class="col span-4">
          <LabeledInput
            v-model="datacenterName"
            :mode="mode"
            :disabled="busy"
            label="Datacenter Name"
          />
          <p class="help-block">String. If you want to use an existing IONOS Datacenter to host this VM, you can change the name here. Please note that if the ID is set it will the prioritized.</p>
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-4">
          <LabeledInput
            v-model="lanId"
            :mode="mode"
            :disabled="busy"
            label="LAN ID"
          />
          <p class="help-block">Optional, integer. The LAN the VM will attach to. If blank, a default LAN will be created. Overrides "Private LAN" setting.</p>
        </div>
        <div class="col span-4">
          <LabeledInput
            v-model="lanName"
            :mode="mode"
            :disabled="busy"
            label="LAN Name"
          />
          <p class="help-block">String. If you want to use an existing IONOS LAN, you can change the name here. Please note that if the ID is set it will the prioritized.</p>
        </div>
        <div class="col span-4">
          <Checkbox
            label="Make Default LAN Private"
            v-model="privateLan"
            :mode="mode"
            :disabled="busy"
          />
          <p class="help-block">If the default LAN does not exist, create a private LAN</p>
        </div>
      </div>

      <div class="row mt-10">
        <div class="col span-4">
          <Checkbox
            label="NIC DHCP"
            v-model="nicDhcp"
            :mode="mode"
            :disabled="busy"
          />
          <p class="help-block">Set whether the created NIC should have dhcp set on or off</p>
        </div>
        <div class="col span-4">
          <StringList
            label="NIC Ips"
            v-model="nicIps"
            :items="nicIps"
            :mode="mode"
            :disabled="busy"
            @change="onChangeNicIps($event)"
          />
          <p class="help-block">Optional. IPBlock reserved IPs. If not set, the driver will reserve an IPBlock automatically or let the API set a private IP if the LAN is private</p>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
  .help-block {
    margin-top: .5em;
    font-size: .8em;
    margin-left: 1em;
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
