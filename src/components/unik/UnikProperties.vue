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
import { VERIFIED_URL_KEY_PREFIX, BADGE_PIONEER_KEY, PioneerBadgeGrades } from "@uns/ts-sdk";

const urlVerifiedRegex = new RegExp(`^${VERIFIED_URL_KEY_PREFIX}[^/]+$`);
const urlVerifiedProofRegex = new RegExp(`^${VERIFIED_URL_KEY_PREFIX}.+/proof$`);

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

  private getValue(entry: property): string {
    if (entry.key === BADGE_PIONEER_KEY) {
      return entry.value === PioneerBadgeGrades.INNOVATOR.toString()
        ? this.$t(`PROPERTIES.${BADGE_PIONEER_KEY}.innovator`).toString()
        : this.$t(`PROPERTIES.${BADGE_PIONEER_KEY}.early`).toString();
    }
    return this.$te(`PROPERTIES.${entry.key}.value`)
      ? this.$t(`PROPERTIES.${entry.key}.value`).toString()
      : entry.value;
  }

  private getKey(entry: property): string {
    if (urlVerifiedRegex.test(entry.key)) {
      return this.$t("PROPERTIES.VERIFIED_URL").toString();
    }
    if (urlVerifiedProofRegex.test(entry.key)) {
      return this.$t("PROPERTIES.VERIFIED_URL_PROOF").toString();
    }
    return this.$te(`PROPERTIES.${entry.key}.key`) ? this.$t(`PROPERTIES.${entry.key}.key`).toString() : entry.key;
  }
}
</script>
