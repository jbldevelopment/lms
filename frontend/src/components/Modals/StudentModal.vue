<template>
	<Dialog
		v-model="show"
		:options="{
			title: __('Add a Student'),
			size: 'sm',
			actions: [
				{
					label: 'Submit',
					variant: 'solid',
					onClick: (close) => addStudent(close),
				},
			],
		}"
	>
		<template #body-content>
			<div class="flex flex-col gap-4 mb-5">
				<Link
					doctype="Department"
					v-model="department"
					placeholder="Department"
				/>
			</div>
			<label class="block text-base text-center mb-5 text-gray-600">
				{{ __('OR') }}
			</label>
			<div class="flex flex-col gap-4">
				<Link
					doctype="User"
					v-model="student"
					placeholder="Student"
					:filters="{ ignore_user_type: 1 }"
				/>
			</div>
		</template>
	</Dialog>
</template>
<script setup>
import { Dialog, createResource } from 'frappe-ui'
import { ref } from 'vue'
import Link from '@/components/Controls/Link.vue'

const students = defineModel('reloadStudents')
const department = ref()
const student = ref()
const show = defineModel()

const props = defineProps({
	batch: {
		type: String,
		default: null,
	},
})

const studentResource = createResource({
	url: 'frappe.client.insert',
	makeParams(values) {
		return {
			doc: {
				doctype: 'Batch Student',
				parent: props.batch,
				parenttype: 'LMS Batch',
				parentfield: 'students',
				student: values.student,
			},
		}
	},
})

const getStudents = createResource({
	url: 'frappe.client.get_list',
	makeParams(values) {
		return {
			doctype: 'Employee',
			filters: {
				department: department.value,
			},
			fields: ['user_id'],
		}
	},
	transform(data) {
		data.forEach((row, index) => {
			studentResource.submit(
				{
					student: row.user_id,
				},
				{
					onSuccess() {
						students.value.reload()
						department.value = null
						student.value = null
					},
				}
			)
		})
		close()
	},
	onSuccess() {
		students.value.reload()
		close()
	},
})

const addStudent = (close) => {
	if (department.value == null) {
		studentResource.submit(
			{
				student: student.value,
			},
			{
				onSuccess() {
					students.value.reload()
					close()
					student.value = null
				},
			}
		)
	} else {
		getStudents.submit()
	}
}
</script>
