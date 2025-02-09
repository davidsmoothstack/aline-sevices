include .env
init: create_cluster create_node_group create_fargate_profile lbInstall
lbInit: create_iam_policy
lbInstall: set_context create_service_account apply_load_balancer

create_cluster:
	eksctl create cluster \
		--name ${clusterName} \
		--region ${region} \
		--full-ecr-access \
		--alb-ingress-access \
		--without-nodegroup \
		--vpc-nat-mode Single \

set_context:
	eksctl utils write-kubeconfig \
		--region ${region} \
		--cluster ${clusterName} \
		--set-kubeconfig-context

create_node_group:
	eksctl create nodegroup \
		--name ${publicNgName} \
		--cluster ${clusterName} \
		--region ${region} \
		--node-type t2.small \
		--nodes-min 1 \
		--nodes-max 2 \
		--full-ecr-access \
		--alb-ingress-access

	eksctl create nodegroup \
		--name ${privateNgName} \
		--node-private-networking \
		--cluster ${clusterName} \
		--region ${region} \
		--node-type t2.small \
		--nodes-min 1 \
		--nodes-max 2 \
		--full-ecr-access \
		--alb-ingress-access

create_fargate_profile:
	eksctl create fargateprofile \
		--cluster ${clusterName} \
		--region ${region} \
		--name ${fargateProfileName} \
		--namespace ${fargateNamespace}

clean:
	eksctl delete cluster \
		--name ${clusterName} \
		--region ${region} \
		--force

create_iam_policy:
	aws iam create-policy \
		--policy-name ${policyName} \
		--policy-document file://iam_policy.json

create_service_account:
	eksctl utils associate-iam-oidc-provider \
		--region=${region} \
		--cluster=${clusterName} \
		--approve

	eksctl create iamserviceaccount \
		--cluster=${clusterName} \
		--region=${region} \
		--namespace=kube-system \
		--name=${serviceAccountName} \
		--attach-policy-arn=arn:aws:iam::${accountId}:policy/${policyName} \
		--override-existing-serviceaccounts \
		--approve

apply_load_balancer:
	helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
		-n kube-system \
		--set clusterName=${clusterName} \
		--set serviceAccount.create=false \
		--set serviceAccount.name=${serviceAccountName}
