<template>
  <div class="px-5 sm:px-10">
    <div class="list-row-border-t-no-wrap">
      <Loader :data="properties" :title="title">
        <div class="list-row">
          <div class="mr-4">{{ title }}</div>
        </div>
        <div class="py-2">
          <TableWrapper v-bind="$attrs" :columns="columns" :rows="properties">
            <template slot-scope="data">
              <div v-if="data.column.field === 'key'">{{ getKey(data.row) }}</div>
              <div v-else-if="data.column.field === 'value'">{{ getValue(data.row) }}</div>
            </template>
          </TableWrapper>
        </div>
      </Loader>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Prop, Component } from "vue-property-decorator";
import { property } from "./Details.vue";
import { TranslateResult } from "vue-i18n";

@Component
export default class UnikProperties extends Vue {
  @Prop({ required: true, default: [] }) public properties: property[];
  @Prop({ required: true, default: [] }) public title: string;

  get columns() {
    const columns = [
      {
        label: this.$t("UNIK.KEY"),
        field: "key",
        thClass: "text-right w-64",
        tdClass: "text-right w-64",
      },
      {
        label: this.$t("UNIK.VALUE"),
        field: "value",
        thClass: "text-left",
        tdClass: "text-left",
      },
    ];
    return columns;
  }

  private getValue(entry: property): string | TranslateResult {
    switch (entry.key) {
      case "Badges/Security/SecondPassphrase":
        return this.$t("BADGES.SECOND_PASSPHRASE_VALUE");
      case "Badges/NP/Delegate":
        return this.$t("BADGES.NETWORKPLAYER_VALUE");
      case "Badges/Pioneer":
        return entry.value === "1" ? this.$t("BADGES.PIONEER_INNOVATOR") : this.$t("BADGES.PIONEER_EARLY");
      default:
        return entry.value;
    }
  }

  private getKey(entry: property): string | TranslateResult {
    switch (entry.key) {
      case "Badges/Security/SecondPassphrase":
        return this.$t("BADGES.SECOND_PASSPHRASE_KEY");
      case "Badges/NP/Delegate":
        return this.$t("BADGES.NETWORKPLAYER_KEY");
      case "Badges/Pioneer":
        return this.$t("BADGES.PIONEER_KEY");
      default:
        return entry.key;
    }
  }
}
</script>
