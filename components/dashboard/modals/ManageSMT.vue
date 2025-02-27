<template>
  <b-modal id="manageSMTModal" title="Update SMT Config" size="lg" centered>
    <template v-if="loading">
      <loading small />
    </template>

    <template v-else>
      <b-row align-h="center">
        <b-col cols="12" lg="6">
          <b-form-group label="Post Reward Curve" description="Post reward curve. Options are: power (r^a).">
            <b-form-select v-model="config.postRewardCurve" :options="postRewardCurveOptions" :state="$v.config.postRewardCurve.$dirty ? !$v.config.postRewardCurve.$error : null" />
          </b-form-group>

          <b-form-group label="Post Reward Curve Parameter" description="For 'power' curve, this is the exponent - 'a' of r^a. Use 1 for linear. Maximum 2 decimal precision.">
            <b-form-input v-model="config.postRewardCurveParameter" :state="$v.config.postRewardCurveParameter.$dirty ? !$v.config.postRewardCurveParameter.$error : null" number />
          </b-form-group>

          <b-form-group label="Curation Reward Curve" description="Curation reward curve. Options are: power (r^a).">
            <b-form-select v-model="config.curationRewardCurve" :options="curationRewardCurveOptions" :state="$v.config.curationRewardCurve.$dirty ? !$v.config.curationRewardCurve.$error : null" />
          </b-form-group>

          <b-form-group label="Curation Reward Curve Parameter" description="For 'power' curve, this is the exponent - 'a' of r^a. Use 1 for linear. Maximum 2 decimal precision.">
            <b-form-input v-model="config.curationRewardCurveParameter" :state="$v.config.curationRewardCurveParameter.$dirty ? !$v.config.curationRewardCurveParameter.$error : null" number />
          </b-form-group>

          <b-form-group label="Curation Reward Percentage" description="What percentage of post reward should be allocated to curators. Must be between 0 and 100.">
            <b-form-input
              v-model="config.curationRewardPercentage"
              :state="$v.config.curationRewardPercentage.$dirty ? !$v.config.curationRewardPercentage.$error : null"
              number
              type="number"
              step="1"
              min="0"
              max="100"
            />
          </b-form-group>

          <b-form-group label="Cashout Window Days" description="How long, in days, until a post is scheduled to pay out. Must be between 1 and 30.">
            <b-form-input
              v-model="config.cashoutWindowDays"
              :state="$v.config.cashoutWindowDays.$dirty ? !$v.config.cashoutWindowDays.$error : null"
              number
              type="number"
              step="1"
              min="0"
              max="30"
            />
          </b-form-group>

          <b-form-group label="Staked Reward Percentage" description="What percentage of rewards should be given as staked. Should be between 0 and 100.">
            <b-form-input
              v-model="config.stakedRewardPercentage"
              :state="$v.config.stakedRewardPercentage.$dirty ? !$v.config.stakedRewardPercentage.$error : null"
              number
              type="number"
              step="1"
              min="0"
              max="100"
            />
          </b-form-group>
        </b-col>

        <b-col cols="12" lg="6">
          <b-form-group label="Reward Interval Seconds" description="How often to add tokens to the reward pool. Must be between 30 to 86400 seconds and divisible by 3.">
            <b-form-input v-model="config.rewardIntervalSeconds" :state="$v.config.rewardIntervalSeconds.$dirty ? !$v.config.rewardIntervalSeconds.$error : null" number />
          </b-form-group>

          <b-form-group label="Reward Per Interval" description="How much to add to the reward pool every reward interval.">
            <b-form-input v-model="config.rewardPerInterval" :state="$v.config.rewardPerInterval.$dirty ? !$v.config.rewardPerInterval.$error : null" number />
          </b-form-group>

          <b-form-group label="Vote Regeneration Days" description="How long it takes to fully regenerate voting power from 0 to 100%.">
            <b-form-input
              v-model="config.voteRegenerationDays"
              :state="$v.config.voteRegenerationDays.$dirty ? !$v.config.voteRegenerationDays.$error : null"
              number
              type="number"
              step="1"
              min="1"
            />
          </b-form-group>

          <b-form-group label="Vote Power Consumption" description="How much vote power is consumed at full voting power for a 100% vote.">
            <b-form-input
              v-model="config.votePowerConsumption"
              :state="$v.config.votePowerConsumption.$dirty ? !$v.config.votePowerConsumption.$error : null"
              number
              type="number"
              step="1"
              min="1"
            />
          </b-form-group>

          <b-form-group label="Downvote Power Regeneration Days" description="How long it takes to fully regenerate downvoting power from 0 to 100%.">
            <b-form-input
              v-model="config.downvoteRegenerationDays"
              :state="$v.config.downvoteRegenerationDays.$dirty ? !$v.config.downvoteRegenerationDays.$error : null"
              number
              type="number"
              step="1"
              min="1"
            />
          </b-form-group>

          <b-form-group label="Downvote Power Consumption" description="How much downvote power is consumed at full voting power for a 100% downvote.">
            <b-form-input
              v-model="config.downvotePowerConsumption"
              :state="$v.config.downvotePowerConsumption.$dirty ? !$v.config.downvotePowerConsumption.$error : null"
              number
              type="number"
              step="1"
              min="1"
            />
          </b-form-group>

          <b-form-group label="Tags" description="Which tags (including community) should be looked at to index a post for this reward pool.">
            <b-form-tags
              v-model="config.tags"
              tag-variant="primary"
              input-id="tags"
              separator=" ,;"
              placeholder="Tags"
              :tag-validator="tagValidator"
            />
          </b-form-group>
        </b-col>
      </b-row>

      <div class="text-info">
        <p>SMT update fee: {{ updateFee }} BEE</p>
        <p>Your current balance: {{ tokenBalance }} BEE</p>
      </div>
    </template>

    <template #modal-footer>
      <b-button variant="primary" :disabled="loading || modalBusy || updateFee > tokenBalance" @click.prevent="requestManageRewardPool">
        <b-spinner v-if="modalBusy" small /> Update
      </b-button>
    </template>
  </b-modal>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { required, numeric, decimal, between } from 'vuelidate/lib/validators'

export default {
  name: 'ManageSMT',

  data () {
    return {
      postRewardCurveOptions: [
        { value: 'power', text: 'Power (r^a)' }
      ],
      curationRewardCurveOptions: [
        { value: 'power', text: 'Power (r^a)' }
      ],

      rewardPoolId: '',
      config: {},

      tokenBalance: 0,
      updateFee: 0,
      modalBusy: false
    }
  },

  computed: {
    ...mapGetters('dashboard', ['smt'])
  },

  mounted () {
    this.$root.$on('bv::modal::show', this.onModalShow)
    this.$root.$on('bv::modal::hidden', this.onModalHidden)

    this.$eventBus.$on('transaction-broadcast-error', this.resetModalBusyState)
  },

  beforeDestroy () {
    this.$root.$off('bv::modal::show', this.onModalShow)
    this.$root.$off('bv::modal::hidden', this.onModalHidden)

    this.$eventBus.$off('transaction-broadcast-error', this.resetModalBusyState)
  },

  methods: {
    ...mapActions(['requestBroadcastJson']),

    requestManageRewardPool () {
      this.$v.config.$touch()

      if (!this.$v.config.$invalid) {
        this.modalBusy = true

        const message = 'Update Reward Pool'
        const eventName = 'smt-reward-pool-update-successful'

        const contractPayload = {
          rewardPoolId: this.rewardPoolId,
          config: this.config
        }

        contractPayload.config.postRewardCurveParameter = this.config.postRewardCurveParameter.toString()
        contractPayload.config.curationRewardCurveParameter = this.config.curationRewardCurveParameter.toString()
        contractPayload.config.rewardPerInterval = this.config.rewardPerInterval.toString()

        const json = {
          contractName: 'comments',
          contractAction: 'updateRewardPool',
          contractPayload
        }

        const jsonData = {
          id: this.$config.SIDECHAIN_ID,
          keyType: 'Active',
          json,
          message,
          eventName
        }

        this.requestBroadcastJson(jsonData)
      }
    },

    async onModalShow (btnEvent, modalId) {
      if (modalId === 'manageSMTModal') {
        this.loading = true

        this.rewardPoolId = this.smt._id
        this.config = JSON.parse(JSON.stringify(this.smt.config))

        const [{ updateFee }, tokenBalance] = await Promise.all([
          this.$sidechain.getContractParams('comments'),
          this.$sidechain.getBalance(this.$auth.user.username, 'BEE')
        ])

        this.updateFee = Number(updateFee)
        this.tokenBalance = tokenBalance ? Number(tokenBalance.balance) : 0

        this.loading = false
      }
    },

    onModalHidden (btnEvent, modalId) {
      if (modalId === 'manageSMTModal') {
        this.$v.$reset()

        this.rewardPoolId = ''
        this.config = {}

        this.modalBusy = false
        this.loading = false
      }
    },

    resetModalBusyState () {
      this.modalBusy = false
    },

    tagValidator (tag) {
      return (
        tag === tag.toLowerCase() &&
        tag.length > 1 &&
        tag.length < 24 &&
        /^[a-z][a-z-0-9]+[a-z0-9]$/.test(tag)
      )
    }
  },

  validations: {
    config: {
      postRewardCurve: {
        required
      },

      postRewardCurveParameter: {
        required,
        validateDecimalPrecision: (v) => {
          if (v === '') { return true }

          return /^\d*(\.\d{0,2})?$/.test(v)
        }
      },

      curationRewardCurve: {
        required
      },

      curationRewardCurveParameter: {
        required,
        validateDecimalPrecision: (v) => {
          if (v === '') { return true }

          return /^\d*(\.\d{0,2})?$/.test(v)
        }
      },

      curationRewardPercentage: {
        required,
        numeric,
        between: between(0, 100)
      },

      cashoutWindowDays: {
        required,
        numeric,
        between: between(0, 100)
      },

      rewardPerInterval: {
        required,
        decimal
      },

      rewardIntervalSeconds: {
        required,
        numeric,
        between: between(30, 86400),
        validInput: (value) => {
          if (value === '') { return true }

          return value % 3 === 0
        }
      },

      voteRegenerationDays: {
        required,
        numeric
      },

      downvoteRegenerationDays: {
        required,
        numeric
      },

      stakedRewardPercentage: {
        required,
        numeric
      },

      votePowerConsumption: {
        required,
        numeric
      },

      downvotePowerConsumption: {
        required,
        numeric
      },

      tags: {
        required
      }
    }
  }
}
</script>

<style>

</style>
