<template>
    <v-date-table
        :headers="headers"
        :items="items"
        item-key="useId"
        no-data-text="データがありません。"
        hide-default-footer
        disable-sort
    >
        <template v-slot:item.approvalStatus="{ item }">
            <v-cip
                :color="
                    &getApprovalStatusColorForWorkflow(
                        item.approval.approvalStatus,
                        approvals[approvals.length - 1].approvalStatus
                    )
                "
                text-color="white"
            >
                {{ item.approval.approvalStatusName }}
            </v-cip>
        </template>
        <template v-slot:item.icon="{item}">
            <ProfileImage :size="40" :user-id="item.userId" :name-kanji="item.userKanji" />
        </template>
        <template #item.score="{item}">
            <div v-if="canEdit(item.userId) && scoreFormat.type === $SCORE_TYPE.RANK.VALUE">
                <v-select
                    :value="item.score"
                    :items="rankValues"
                    class="rank-select"
                    data-test-id="total-hr-evaluation-table-total-rank"
                    :error-massage="
                        $safe(validationError).error[
                            totalEvaluationDetails[${totalEvaluationDetails.findIndex((x) => {
                                return x.userId === editUserId
                            })}]_score
                        ].unsafe
                    "
                    @change="inputTotalRank"
                >
                    <template #item="{item}">
                        <span
                            data-testid="total-hr-evaluation-table-total-rank-item"
                        >
                            {{ item }}
                        </span>
                    </template>
                </v-select>
            </div>
            <span v-else>
                {{ item.score }}
            </span>
        </template>
        <template #item.comment="{item}">
            <v-edit-dialog v-if="canEdit(item.userId)">
                <div class="py-4 mr-4" data-testid="total-hr-evaluation-table-comment">
                    <div class="editable-comment-cell">
                        <v-textarea 
                            rows="1"
                            auto-grow
                            readonly
                            :value="item.comment"
                            :error-messages="
                                $safe(validationError).error[
                                    totalEvaluationDetails[${totalEvaluationDetails.findIndex((x) => {
                                        return x.userId === editUserId
                                    })}]_comment
                                ].unsafe
                            "
                        />
                        <v-icon class="editable-comment-cell__icon" color="primary">
                            mdi-pencil-box
                        </v-icon>
                    </div>
                </div>
                <template v-slot:input>
                    <v-textarea
                        rows="3"
                        :value="item.comment"
                        data-testid="total-hr-evaluation-table-comment-textarea"
                        @change="inputTotalComment"
                    />
                </template>
            </v-edit-dialog>
            <div v-else class="py-2 mr-4">
                <span>{{ item.comment }}</span>
            </div>
        </template>
    </v-date-table>
</template>

<script>
export default{
    props: {
        initialTotalEvaluationDetails: {
            type: Array,
            default(){
                return{}
            },
        },
        scoreFormat: {
            type: Object,
            default(){
                return{}
            },
        },
        workflows: {
            type: Array,
            default(){
                return{}
            },
        },
        targetUserId: {
            type: String,
            default(){
                return{}
            },
        },
        validationError: {
            type: Object,
            default(){
                return{}
            },
        },
        editUserId: {
            type: Object,
            default(){
                return{}
            },
        },
        propsCanEdit: {
            type: Boolean,
            default(){
                return false
            },
        },
        approvals: {
            type: Array,
            default(){
                return []
            },
        },
    },
    data() {
        return {
            totalEvaluationDetails: this.initialTotalEvaluationDetails,
        }
    },
    computed: {
        headers(){
            const results = []
            results.push({text: '', value: 'workflow', width: '90'})
            if(this.approvals.length){
                results.push({text: 'ステータス', value: 'approvalStatus', width: '90'})
            }
            results.push({ text:'', value: 'icon', width: '40' })
            results.push({ text:'氏名', value: 'userNameKanji', width: '130' })
            results.push({ text:'会社名', value: 'userMainCompanyName', width: '130' })
            results.push({ text:'部署名', value: 'userMainDivisionName', width: '130' })
            results.push({ text:'評価', value: 'score', width: '90' })
            results.push({ text:'コメント', value: 'comment', width: '340' })
            return results
        },
        items(){
            return this.workflows.map((workflow) => {
                return {
                    workflow: this.$getWorkflowStepText(workflow.order, this.workflows),
                    ...workflow,
                    score: this.getTotalEvaluationDetailScore(workflow.userId),
                    comment: this.getTotalEvaluationDetailComment(workflow.userId),
                    approval: this.getApproval(workflow.userId),
                }
            })
        },
        rankValues(){
            return this.scoreFormat.rankValue.split(',')
        },
    },
    watch: {
        initialTotalEvaluationDetails: {
            handler(details){
                this.totalEvaluationDetails = details
            },
            deep: true,
        },
    },
    methods: {
        canEdit(userId){
            return this.editUserId === userId && this.propsCanEdit
        },
        inputTotalComment(value){
            this.$emit('input', value)
        },
        inputTotalRank(value){
            this.$emit('input-total-rank', value)
        },
        getTotalEvaluationDetail(userId){
            return this.$Enumerable
                .from(this.totalEvaluationDetails)
                .where((x) => x.userId === userId)
                .firstOrDefault()
        },
        getTotalEvaluationDetailScore(userId){
            const data = this.getTotalEvaluationDetail(userId)
            if(data === undefined){
                return ''
            } else { 
                return data.score
            }
        },
        getTotalEvaluationDetailComment(userId) {
            const data = this.getTotalEvaluationDetail(userId)
            if(data === undefined) {
                return ''
            } else { 
                return data.comment
            }
        },
        getApproval(userId){
            return this.approvals.find((x) => x.userId === userId)
        },
    },
}
</script>

<style lang="scss" scoped>
.editable-comment-cell {
    position: relative;
    &__icon {
        position: absolute;
        right: -24px;
        top: 50%;
        transform: translateY(-50%);
    }
}
.rank-select {
    font-size: 14px;
}
::v-deep .v-small-dialog__activator__content {
    display: block;
}
</style>
