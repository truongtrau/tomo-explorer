<template>
    <div
        v-if="loading"
        :class="(loading ? 'tomo-loading tomo-loading--full' : '')"/>
    <section v-else>
        <div
            v-if="total == 0"
            class="tomo-empty">
            <i class="fa fa-user-secret tomo-empty__icon"/>
            <p class="tomo-empty__description">No account found</p>
        </div>

        <p
            v-if="total > 0"
            class="tomo-total-items">Total {{ _nFormatNumber('account', 'accounts', total) }} found</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="tomo-table--accounts">

            <template
                slot="rank"
                slot-scope="props">{{ props.item.rank }}</template>

            <template
                slot="hash"
                slot-scope="props">
                <div>
                    <i
                        v-if="props.item.isContract"
                        class="tm tm-icon-contract mr-1 mr-lg-2" />
                    <nuxt-link
                        :to="{name: 'address-slug', params: {slug: props.item.hash}}"
                        class="text-truncate">{{ props.item.hash }}</nuxt-link>
                </div>
            </template>

            <template
                slot="balance"
                slot-scope="props">
                <span
                    class="d-lg-none"
                    v-html="formatUnit(toEther(props.item.balance, 5))"/>
                <span
                    class="d-none d-lg-block"
                    v-html="formatUnit(toEther(props.item.balance))"/>
            </template>

            <template
                slot="txCount"
                slot-scope="props">
                <span>{{ formatNumber(props.item.transactionCount) }}</span>
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
    head: () => ({
        title: 'Accounts'
    }),
    data: () => ({
        fields: {
            rank: { label: 'Rank' },
            hash: { label: 'Address' },
            balance: { label: 'Balance' },
            transactionCount: { label: 'TxCount' }
        },
        loading: true,
        pagination: {},
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 15,
        pages: 1
    }),
    async mounted () {
        let self = this
        // Init breadcrumbs data.
        this.$store.commit('breadcrumb/setItems', { name: 'accounts', to: { name: 'accounts' } })

        await self.getDataFromApi()
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

            let query = this.serializeQuery(params)
            let { data } = await this.$axios.get('/api/accounts' + '?' + query)
            self.items = data.items
            self.total = data.total
            self.currentPage = data.currentPage
            self.pages = data.pages

            // Hide loading.
            self.loading = false

            return data
        },

        onChangePaginate (page) {
            let self = this
            self.currentPage = page

            self.getDataFromApi()
        },

        linkGen (pageNumber) {
            let self = this
            let query = {
                page: pageNumber,
                limit: self.perPage
            }

            return {
                name: 'accounts',
                query: query
            }
        }
    }
}
</script>

<style scoped>
</style>
