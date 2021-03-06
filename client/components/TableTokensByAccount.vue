<template>
    <div
        v-if="loading"
        :class="(loading ? 'tomo-loading tomo-loading--full' : '')"/>
    <section v-else>
        <div
            v-if="total == 0"
            class="tomo-empty">
            <i class="fa fa-exchange tomo-empty__icon"/>
            <p class="tomo-empty__description">No token found</p>
        </div>

        <p
            v-if="total > 0"
            class="tomo-total-items">Total {{ _nFormatNumber('token', 'tokens', total) }} found</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="tomo-table--tokens-by-account">

            <template
                slot="hash"
                slot-scope="props">
                <nuxt-link
                    :class="props.item.tokenObj ? '' : 'text-truncate'"
                    :to="{name: 'tokens-slug', params: {slug: props.item.token}}">
                    {{ props.item.tokenObj ? props.item.tokenObj.name : props.item.token }}</nuxt-link>
            </template>

            <template
                slot="quantity"
                slot-scope="props">
                {{ formatUnit(toEther(convertHexToFloat(props.item.quantity, 16)), props.item.tokenObj ?
                props.item.tokenObj.symbol : '') }}
            </template>

        </table-base>

        <b-pagination
            v-if="total > 0 && total > perPage"
            v-model="currentPage"
            :total-rows="total"
            :per-page="perPage"
            align="center"
            class="tomo-pagination"
            @change="onChangePaginate"
        />
    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import TableBase from '~/components/TableBase'

export default {
    components: {
        TableBase
    },
    mixins: [mixin],
    props: {
        address: {
            type: String,
            default: ''
        },
        page: {
            type: Object,
            default: () => {
                return {}
            }
        }
    },
    data: () => ({
        fields: {
            hash: { label: 'Token' },
            quantity: { label: 'Quantity' }
        },
        loading: true,
        pagination: {},
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 15,
        pages: 1,
        block: null
    }),
    async mounted () {
        this.getDataFromApi()
    },
    methods: {
        async getDataFromApi () {
            let self = this

            // Show loading.
            self.loading = true

            let params = {
                page: self.currentPage,
                limit: self.perPage
            }

            if (self.address) {
                params.hash = self.address
            }

            let query = this.serializeQuery(params)
            let { data } = await this.$axios.get('/api/token-holders' + '?' + query)
            self.items = data.items
            self.total = data.total
            self.currentPage = data.currentPage
            self.pages = data.pages

            if (self.page) {
                self.page.tokensCount = self.total
            }

            // Hide loading.
            self.loading = false

            return data
        },
        onChangePaginate (page) {
            let self = this
            self.currentPage = page

            self.getDataFromApi()
        }
    }
}
</script>
