<template>
    <div>
        <b-list-group flush>
            <ListItem>
                <template slot="label">Status</template>
                <template slot="item-content">
                    <div class="d-flex align-items-center">
                        <b-badge
                            v-if="tx.reverted"
                            style="background-color: red"
                            class="mr-2"
                        >Reverted</b-badge>
                        <b-badge v-else class="mr-2" variant="success">Success</b-badge>
                        <span>{{txStatus}}</span>
                    </div>
                    <div v-if="tx.reverted">
                        <b-icon icon="arrow-return-right"/> <small>{{tx.vmError.reason ? tx.vmError.reason : tx.vmError.error}}</small>
                    </div>
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">ID</template>
                <template slot="item-content">
                    <span class="text-monospace font-weight-lighter">{{tx.txID}}</span>
                    <b-tooltip :triggers="[]" target="tx-id-btn" ref="tx-id-btn-tip">Copied</b-tooltip>
                    <b-button
                        id="tx-id-btn"
                        v-clipboard:copy="tx.txID"
                        v-clipboard:success="onCopy"
                        class="ml-1 ml-sm-3"
                        size="sm"
                        variant="outline-secondary"
                    >
                        <font-awesome-icon small icon="copy" />
                    </b-button>
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">Timestamp</template>
                <template slot="item-content">{{tx.blockTimestamp | datetime}}</template>
            </ListItem>
            <ListItem>
                <template slot="label">Clauses</template>
                <template slot="item-content">
                    <b-button
                        to="#clauses"
                        size="sm"
                        variant="outline-primary"
                    >{{tx.clauses.length}}</b-button>
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">Total Transfer</template>
                <template slot="item-content">
                    <Amount :amount="tx.clauses | countVal" sym="VET" />
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">Gas Used</template>
                <template slot="item-content">
                    <span class="text-monospace">{{tx.gasUsed | numFmt}}/{{tx.gas | numFmt}}</span>
                    <sup>
                        <strong>price coef {{tx.gasPriceCoef}}</strong>
                    </sup>
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">Origin</template>
                <template slot="item-content">
                    <AccountLink :address="tx.origin" :short="false" />
                </template>
            </ListItem>
            <ListItem>
                <template slot="label">Fee</template>
                <template slot="item-content">
                    <div class="d-inline-block d-sm-flex align-items-center">
                        <Amount :amount="tx.paid" sym="VTHO" />
                        <template v-if="tx.delegator">
                            <span class="mx-2">delegated by</span>
                            <AccountLink :address="tx.delegator" size="sm" />
                        </template>
                        <template v-else-if="tx.gasPayer !== tx.origin">
                            <span class="mx-2">sponsored by</span>
                            <AccountLink :address="tx.gasPayer" size="sm" />
                        </template>
                        <template v-else>
                            <span class="mx-2">paid by</span>
                            <strong>Origin</strong>
                        </template>
                    </div>
                </template>
            </ListItem>
            <ListItem v-if="contractAddr.length">
                <template slot="label">Contract Creation</template>
                <template slot="item-content">
                    <ul class="mb-0 pl-0">
                        <li
                            class="pt-1 pb-2 d-inline-block d-sm-flex align-items-center"
                            v-for="(item, i) in contractAddr"
                            :key="i"
                        >
                            <b-link
                                class="d-inline-block d-sm-flex mr-1"
                                :href="'#clauses-' + (item.index + 1)"
                            >
                                <small>@clause#{{item.index + 1}}</small>
                            </b-link>
                            <small class="mx-2">contract</small>
                            <AccountLink :address="item.address" size="sm" />
                            <small class="mx-2">created by</small>
                            <strong>Origin</strong>
                        </li>
                    </ul>
                </template>
            </ListItem>
            <ListItem v-if="transfersList.length">
                <template slot="label">Token Transferred</template>
                <template slot="item-content">
                    <ul class="mb-0 pl-0">
                        <li
                            class="pt-1 pb-2 d-inline-block d-sm-flex align-items-center"
                            v-for="(item, i) in transfersList"
                            :key="i"
                        >
                            <b-link
                                class="d-inline-block d-sm-flex mr-1"
                                :href="'#clauses-' + (item.clauseIndex + 1)"
                            >
                                <small>@clause#{{item.clauseIndex + 1}}</small>
                            </b-link>
                            <TokenTransferItem :origin="tx.origin" :transfer="item" />
                        </li>
                    </ul>
                </template>
            </ListItem>

            <template v-if="isMore">
                <ListItem>
                    <template slot="label">Size</template>
                    <template slot="item-content">{{tx.size}} B</template>
                </ListItem>
                <ListItem>
                    <template slot="label">Reward</template>
                    <template slot="item-content">
                        <Amount :amount="tx.reward" sym="VTHO" />
                    </template>
                </ListItem>
                <ListItem>
                    <template slot="label">BlockRef</template>
                    <template slot="item-content">
                        <span class="text-monospace font-weight-lighter">
                            <nuxt-link
                                :to="{name: 'search', query: {content: blcokRefNum(tx.blockRef)}}"
                            >{{tx.blockRef | bNum | numFmt}}</nuxt-link>
                        </span>
                        <span class="text-monospace font-weight-lighter">({{tx.blockRef}})</span>
                    </template>
                </ListItem>
                <ListItem>
                    <template slot="label">Expiration</template>
                    <template slot="item-content">{{tx.expiration + ' blocks'}}</template>
                </ListItem>
                <ListItem>
                    <template slot="label">Nonce</template>
                    <template slot="item-content">
                        <span class="text-monospace font-weight-lighter">{{tx.nonce}}</span>
                    </template>
                </ListItem>
                <ListItem>
                    <template slot="label">Depends On</template>
                    <template slot="item-content">
                        <TxLink :short="false" v-if="tx.dependsOn" :id="tx.dependsOn" />
                        <span v-else>-</span>
                    </template>
                </ListItem>
                <ListItem>
                    <template slot="label">Chain Tag</template>
                    <template slot="item-content">
                        <span
                            class="text-monospace font-weight-lighter"
                        >0x{{tx.chainTag.toString(16)}}</span>
                    </template>
                </ListItem>
            </template>
            <b-list-group-item class="pl-1">
                <b-button
                    size="sm"
                    variant="link"
                    @click="isMore = !isMore"
                >View {{ isMore ? 'less' : 'more'}}</b-button>
            </b-list-group-item>
        </b-list-group>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'
import ListItem from '@/components/ListItem.vue'
import IdentBox from '@/components/IdentBox.vue'
import AccountLink from '@/components/AccountLink.vue'
import TxLink from '@/components/TxLink.vue'
import TokenTransferItem from '@/components/TokenTransferItem.vue'
import Amount from '@/components/Amount.vue'
import { Context } from '@nuxt/types'
@Component({
    components: {
        ListItem,
        IdentBox,
        TokenTransferItem,
        AccountLink,
        Amount,
        TxLink
    }
})
export default class TxInfo extends Vue {
    @Prop(Object)
    tx!: DTO.Tx & DTO.BlockMeta & { outputs: DTO.Output[] } & DTO.Receipt

    @Prop(Number)
    bestNum!: number

    @Prop()
    transfers!: DTO.Transfer[]

    showTip = false
    isMore = false
    get txStatus() {
        const confirm = this.bestNum - this.tx.blockNumber
        return ` ${confirm > 12 ? '≥ 12' : Math.max(0, confirm)} Confirmations`
    }

    get transfersList() {
        return this.transfers.map(item => {
            return {
                sender: item.sender,
                clauseIndex: item.meta.clauseIndex,
                recipient: item.recipient,
                amount: item.amount,
                symbol: item.symbol,
                decimals: item.decimals
            }
        })
    }

    get contractAddr() {
        const result: { address: string; index: number }[] = []
        this.tx.outputs.forEach((item, i) => {
            if (item.contractAddress) {
                result.push({
                    address: item.contractAddress,
                    index: i
                })
            }
        })
        return result
    }

    blcokRefNum(blockRef: string) {
        return Vue.filter('bNum')(blockRef)
    }

    onCopy() {
        const t = this.$refs['tx-id-btn-tip'] as Vue
        t.$emit('open')
        setTimeout(() => {
            t.$emit('close')
        }, 1000)
    }
}
</script>


