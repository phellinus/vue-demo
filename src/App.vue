<script setup lang="ts">
import { computed, reactive, ref } from "vue";
import { ElMessage, type FormInstance, type FormRules } from "element-plus";

interface RuleForm {
	deviceType: "" | "camera" | "phone";
	brand: string;
	price: number | null;
	remark: string;

	lensConfig: string;
	phoneModel: string;
	focusRange: string;
}

type DynamicFieldProp = "lensConfig" | "phoneModel" | "focusRange";

interface DynamicField {
	label: string;
	prop: DynamicFieldProp;
	type: "input";
	placeholder: string;
	rules: {
		required?: boolean;
		message: string;
		trigger: "blur" | "change";
	}[];
}

const formRef = ref<FormInstance>();

const ruleForm = reactive<RuleForm>({
	deviceType: "",
	brand: "",
	price: null,
	remark: "",

	lensConfig: "",
	phoneModel: "",
	focusRange: "",
});

const fieldMap: Record<"camera" | "phone", DynamicField[]> = {
	camera: [
		{
			label: "镜头配置",
			prop: "lensConfig",
			type: "input",
			placeholder: "请输入镜头配置",
			rules: [
				{
					required: true,
					message: "请输入镜头配置",
					trigger: "blur",
				},
			],
		},
		{
			label: "聚焦范围",
			prop: "focusRange",
			type: "input",
			placeholder: "请输入聚焦范围",
			rules: [
				{
					required: true,
					message: "请输入聚焦范围",
					trigger: "blur",
				},
			],
		},
	],
	phone: [
		{
			label: "手机型号",
			prop: "phoneModel",
			type: "input",
			placeholder: "请输入手机型号",
			rules: [
				{
					required: true,
					message: "请输入手机型号",
					trigger: "blur",
				},
			],
		},
	],
};

const currentFields = computed<DynamicField[]>(() => {
	if (!ruleForm.deviceType) return [];
	return fieldMap[ruleForm.deviceType] || [];
});
/**
 * 设备变化时，清除隐藏字段的校验状态
 * 这样切换后不会残留上一个字段的报错
 */
function handleDeviceChange() {
	formRef.value?.clearValidate(["lensConfig", "phoneModel", "focusRange"]);
}

const baseRules: FormRules = {
	deviceType: [{ required: true, message: "请选择设备", trigger: "change" }],
	brand: [{ required: true, message: "请输入品牌", trigger: "blur" }],
	price: [{ required: true, message: "请输入价格", trigger: "change" }],
};

const formRules = computed<FormRules>(() => {
	const rules: FormRules = {
		...baseRules,
	};

	currentFields.value.forEach((field) => {
		rules[field.prop] = field.rules;
	});

	return rules;
});

function buildSubmitData() {
	const data: Partial<RuleForm> = {
		deviceType: ruleForm.deviceType,
		brand: ruleForm.brand,
		price: ruleForm.price,
		remark: ruleForm.remark,
	};

	currentFields.value.forEach((field) => {
		data[field.prop] = ruleForm[field.prop];
	});

	return data;
}

async function handleSubmit() {
	if (!formRef.value) return;

	try {
		await formRef.value.validate();
		const submitData = buildSubmitData();
		console.log("提交数据:", submitData);
		ElMessage.success("提交成功");
	} catch {
		ElMessage.error("请先完善表单");
	}
}
function handleReset() {
	ruleForm.deviceType = "";
	ruleForm.brand = "";
	ruleForm.price = null;
	ruleForm.remark = "";
	ruleForm.lensConfig = "";
	ruleForm.phoneModel = "";
    ruleForm.focusRange = "";
	formRef.value?.clearValidate();
}
</script>

<template>
	<div style="display: flex; justify-content: center">
		<el-card style="width: 600px">
			<template #header>
				<h3>动态表单提交</h3>
			</template>
			<el-form
				ref="formRef"
				:model="ruleForm"
				:rules="formRules"
				label-width="100px"
				:validate-on-rule-change="false"
			>
				<el-form-item label="设备" prop="deviceType">
					<el-select
						v-model="ruleForm.deviceType"
						placeholder="请选择设备"
						@change="handleDeviceChange"
					>
						<el-option label="相机" value="camera" />
						<el-option label="手机" value="phone" />
					</el-select>
				</el-form-item>

				<el-form-item label="品牌" prop="brand">
					<el-input
						v-model="ruleForm.brand"
						placeholder="请输入品牌"
					/>
				</el-form-item>

				<el-form-item label="价格" prop="price">
					<el-input-number v-model="ruleForm.price" :min="0" />
				</el-form-item>

				<el-form-item label="备注" prop="remark">
					<el-input
						v-model="ruleForm.remark"
						type="textarea"
						placeholder="请输入备注"
					/>
				</el-form-item>

				<!-- 动态字段 -->
				<template v-for="field in currentFields" :key="field.prop">
					<el-form-item :label="field.label" :prop="field.prop">
						<el-input
							v-if="field.type === 'input'"
							v-model="ruleForm[field.prop]"
							:placeholder="field.placeholder"
						/>
					</el-form-item>
				</template>

				<el-form-item>
					<el-button type="primary" @click="handleSubmit"
						>提交</el-button
					>
					<el-button @click="handleReset">重置</el-button>
				</el-form-item>
			</el-form>
		</el-card>
	</div>
</template>
