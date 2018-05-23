<template>
	<section>
		<p class="tm__total">Total {{ formatNumber(total) }} items found</p>

		<b-table class="tm__table"
			small
			:fields="fields"
			:loading="loading"
			:items="items">
			<template slot="transactionHash" slot-scope="props">
				<div class="tm__cell">
					<nuxt-link class="address__tag" :to="{name: 'txs-slug', params: {slug: props.item.transactionHash}}">{{ props.item.transactionHash }}</nuxt-link>
				</div>
			</template>

			<template slot="block" slot-scope="props">
				<div class="tm__cell">
					<nuxt-link v-if="props.item.block" class="address__tag" :to="{name: 'blocks-slug', params: {slug: props.item.blockNumber}}">{{ props.item.blockNumber }}</nuxt-link>
					<span v-else class="text-muted">Pending...</span>
				</div>
			</template>

			<template slot="timestamp" slot-scope="props">
				<div class="tm__cell">
					<div v-if="props.item.timestamp">
						<span :id="'age__' + props.index">{{ $moment(props.item.timestamp).fromNow() }}</span>
						<b-tooltip :target="'age__' + props.index">
							{{ $moment(props.item.timestamp).format('MMM-DD-Y hh:mm:ss A') }}
						</b-tooltip>
					</div>
				</div>
			</template>

			<template slot="from" slot-scope="props">
				<div class="tm__cell">
					<div class="address__tag">
						<i v-if="props.item.from_model && props.item.from_model.isContract" class="tm tm-icon-contract pull-left mr-1"></i>
						<span v-if="address == props.item.from">{{ props.item.from }}</span>
						<nuxt-link v-else :to="{name: 'address-slug', params: {slug: props.item.from}}">{{ props.item.from }}</nuxt-link>
					</div>
				</div>
			</template>

			<template slot="arrow" slot-scope="props">
				<div class="tm__cell">
					<i class="tm-arrow-right" :class="props.item.from == address ? 'text-danger' : 'text-success'"></i>
				</div>
			</template>

			<template slot="to" slot-scope="props">
				<div class="tm__cell">
					<div class="address__tag">
						<i v-if="props.item.to_model && props.item.to_model.isContract" class="tm tm-icon-contract pull-left mr-1"></i>
						<span v-if="address == props.item.to">{{ props.item.to }}</span>
						<nuxt-link v-else :to="{name: 'address-slug', params:{slug: props.item.to}}">
							<span>{{ props.item.to }}</span>
						</nuxt-link>
					</div>
				</div>
			</template>

			<template slot="value" slot-scope="props">
				<div class="tm__cell">
					{{ toEther(props.item.value) }}
				</div>
			</template>

			<template slot="token" slot-scope="props">
				<div class="tm__cell">
					<nuxt-link v-if="props.item.symbol" :to="{name: 'tokens-slug', params: {slug: props.item.address}}">ERC20 ({{ props.item.symbol }})</nuxt-link>
					<i v-else>ERC20</i>
				</div>
			</template>
		</b-table>
		<b-pagination
			align="center"
			:total-rows="total"
			:per-page="perPage"
			@change="onChangePaginate"
		></b-pagination>
	</section>
</template>
<script>
  import mixin from '~/plugins/mixin'

  export default {
    mixins: [mixin],
    props: {
      token: {type: String, default: null},
    },
    data: () => ({
      fields: {
        transactionHash: {label: 'TxHash'},
        timestamp: {label: 'LastSeen'},
        from: {label: 'from'},
        arrow: {class: 'text-center'},
        to: {label: 'To'},
        value: {label: 'Value', class: 'text-right'},
        token: {label: 'Token'},
      },
      loading: true,
      pagination: {},
      total: 0,
      items: [],
      currentPage: 1,
      perPage: 15,
      pages: 1,
      address: null,
    }),
    async mounted () {
      let self = this
      // Init from router.
      let query = self.$route.query
      if (query.page) {
        self.currentPage = parseInt(query.page)
      }
      if (query.limit) {
        self.perPage = parseInt(query.limit)
      }

      if (query.address) {
        self.address = query.address
      }

      this.getDataFromApi()
    },
    methods: {
      async getDataFromApi () {
        let self = this

        // Show loading.
        self.loading = true

        let params = {
          page: self.currentPage,
          limit: self.perPage,
        }

        this.$router.replace({query: params})

        if (self.token) {
          params.token = self.token
        }
        if (self.address) {
          params.address = self.address
        }

        let query = this.serializeQuery(params)
        let {data} = await this.$axios.get('/api/token-txs' + '?' + query)
        self.items = data.items
        self.total = data.total
        self.currentPage = data.currentPage
        self.pages = data.pages

        // Hide loading.
        self.loading = false

        // Format data.
        self.items = self.formatData(self.items)

        return data
      },
      formatData (items = []) {
        let _items = []
        items.forEach((item) => {
          let _item = item

          // Format for timestamp.
          if (!item.block) {
            _item.timestamp = item.createdAt
          }
          else {
            _item.timestamp = item.block.timestamp
          }

          _items.push(_item)
        })

        return _items
      },
      onChangePaginate (page) {
        let self = this
        self.currentPage = page

        self.getDataFromApi()
      },
    },
  }
</script>