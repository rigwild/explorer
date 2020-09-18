<template>
  <span
    :class="
      !isFee && source > 0
        ? {
            'text-red': isOutgoing,
            'text-green': isIncoming,
          }
        : ''
    "
    class="flex items-center whitespace-no-wrap"
  >
    <span
      v-tooltip="{
        trigger: 'hover click',
        content: source && price ? readableCurrency(source, price) : '',
        placement: tooltipPlacement,
      }"
      class="ml-auto"
    >
      <span v-if="!isFee && isIncoming && source > 0">+</span>
      <span v-else-if="!isFee && isOutgoing && source > 0">-</span>
      {{ readableCrypto(source) }}
    </span>
    <SvgIcon
      v-if="showAmountInformation"
      v-tooltip="{
        trigger: 'hover click',
        content: $t('TRANSACTION.AMOUNT_TO_SELF', { amount: readableCrypto(amountToSelf) }),
      }"
      class="ml-2"
      name="information"
      view-box="0 0 16 16"
    />
  </span>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";
import { ITransaction } from "@/interfaces";
import { CoreTransaction, MagistrateTransaction, TypeGroupTransaction } from "@/enums";
import { mapGetters } from "vuex";
import { getMilestone } from "./utils";
import { DIDType, DIDHelpers } from "@uns/ts-sdk";
import { Identities } from "@uns/ark-crypto";

@Component({
  computed: {
    ...mapGetters("network", ["height", "networkConfig"]),
  },
})
export default class TransactionAmount extends Vue {
  @Prop({ required: true }) public transaction: ITransaction;
  @Prop({ required: false, default: false }) public isFee: boolean;
  @Prop({ required: false, default: "top" }) public tooltipPlacement: string;

  private height: number;
  private initialBlockHeight = 0;
  private networkConfig;
  private initialMilestone = null;
  private foundationWallet: string = null;

  @Watch("height")
  public onHeightChanged(newValue: number, oldValue: number) {
    if (!oldValue) {
      this.setInitialBlockHeight();
    }
  }

  private setFoundationWallet() {
    if (!this.foundationWallet) {
      const foundationPubKey = this.networkConfig.network.foundation.publicKey;
      this.foundationWallet = Identities.Address.fromPublicKey(foundationPubKey, this.networkConfig.network.pubKeyHash);
    }
  }

  private setInitialBlockHeight() {
    this.initialBlockHeight = this.height - this.transaction.confirmations;
  }

  private setInitialMilestone() {
    if (!this.initialBlockHeight) {
      this.setInitialBlockHeight();
    }
    this.initialMilestone = getMilestone(this.networkConfig, this.initialBlockHeight);
  }

  get transactionTab() {
    return this.$store.getters["ui/walletTransactionTab"];
  }

  get address() {
    return this.$route.params.address || this.transaction.sender !== this.$route.params.address
      ? this.$route.params.address
      : undefined;
  }

  get showAmountInformation() {
    return (
      !this.isFee && this.transactionTab === "all" && this.address && this.amountToSelf && !this.amountToSelf.isZero()
    );
  }

  get source() {
    if (this.isFee) {
      return this.transaction.fee;
    }

    // @ts-ignore
    if (this.isMultiPayment(this.transaction.type, this.transaction.typeGroup)) {
      // @ts-ignore
      return this.calculateMultipaymentAmount(this.transaction, this.address, this.transactionTab);
    }

    // @ts-ignore
    if (this.isVoucherUnsCertifiedNftMint(this.transaction)) {
      const type: DIDType = DIDHelpers.fromCode(parseInt(this.transaction.asset.nft.unik.properties.type));
      this.setInitialMilestone();
      const rewards = this.initialMilestone.voucherRewards[type.toLowerCase()];

      // Forge factory wallet
      const issuerAddress = this.transaction.recipient;
      if (this.$route.params.address === issuerAddress) {
        return this.transaction.amount;
      }

      // Foundation wallet
      this.setFoundationWallet();
      if (this.$route.params.address === this.foundationWallet) {
        return rewards.foundation;
      }
      return rewards.sender + rewards.forger;
    }

    return this.transaction.amount;
  }

  get amountToSelf() {
    // @ts-ignore
    if (
      this.transaction.sender === this.address &&
      // @ts-ignore
      this.isMultiPayment(this.transaction.type, this.transaction.typeGroup)
    ) {
      // @ts-ignore
      return this.calculateMultipaymentAmount(this.transaction, this.address, "received");
    }
  }

  get price() {
    return this.transaction.price;
  }

  get isOutgoing() {
    // @ts-ignore
    if (this.isTimelock(this.transaction.type, this.transaction.typeGroup)) {
      return (
        (this.$route.params.address !== this.transaction.recipient &&
          this.transaction.lockStatus === CoreTransaction.TIMELOCK_CLAIM) ||
        (this.$route.params.address !== this.transaction.sender &&
          this.transaction.lockStatus === CoreTransaction.TIMELOCK_REFUND)
      );
    }

    // @ts-ignore
    if (this.isUnsUpdateService(this.transaction)) {
      return true;
    }

    return this.transaction.sender === this.$route.params.address;
  }

  get isIncoming() {
    // @ts-ignore
    if (this.isTimelock(this.transaction.type, this.transaction.typeGroup)) {
      return (
        this.$route.params.address !== this.transaction.sender &&
        this.transaction.lockStatus === CoreTransaction.TIMELOCK_CLAIM
      );
    }

    // @ts-ignore
    if (this.isMultiPayment(this.transaction.type, this.transaction.typeGroup)) {
      return (
        this.transaction.asset.payments.find((payment) => payment.recipientId === this.$route.params.address) &&
        (this.transactionTab === "received" || this.transaction.sender !== this.$route.params.address)
      );
    }

    // @ts-ignore
    if (this.isUnsCertifiedNftMint(this.transaction.type, this.transaction.typeGroup)) {
      // @ts-ignore
      if (this.isVoucherUnsCertifiedNftMint(this.transaction)) {
        return true;
      }
      // forge factory case
      if (this.$route.params.address === this.transaction.recipient) {
        return true;
      }
    }

    // @ts-ignore
    if (this.isVoucherUnsCertifiedNftMint(this.transaction)) {
      return true;
    }

    return (
      this.transaction.recipient === this.$route.params.address &&
      // @ts-ignore
      (this.isTransfer(this.transaction.type, this.transaction.typeGroup) ||
        // @ts-ignore
        this.isUnsUpdateService(this.transaction))
    );
  }
}
</script>
