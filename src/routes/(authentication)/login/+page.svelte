<script>
	import Link from 'svelte-link';
	import { _ } from 'svelte-i18n'
	import {
		Row,
		Col,
		CardBody,
		Card,
		Container,
		Form,
		Label,
		Input,
		Button,
		Alert
	} from '@sveltestrap/sveltestrap';
	import Headtitle from '$lib/common/HeadTitle.svelte';
	import { getToken } from '$lib/services/auth-service.js';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import {
		PUBLIC_SERVICE_URL,
		PUBLIC_LIVECHAT_HOST,
		PUBLIC_LOGO_URL,
		PUBLIC_LOGIN_IMAGE,
		PUBLIC_BRAND_NAME,
		PUBLIC_ADMIN_USERNAME,
		PUBLIC_ADMIN_PASSWORD,
		PUBLIC_COMPANY_NAME,
		PUBLIC_ALLOW_SIGNUP,
		PUBLIC_AUTH_ENABLE_SSO,
		PUBLIC_AUTH_ENABLE_FIND_PWD,
	} from '$env/static/public';
	import { onMount } from 'svelte';
	import { resetLocalStorage } from '$lib/helpers/store';

	let username = PUBLIC_ADMIN_USERNAME;
	let password = PUBLIC_ADMIN_PASSWORD;
	let isOpen = false;
	let msg = '';
	let status = '';
	let isSubmitting = false;
	let isRememberMe = false;

	onMount(() => {
		const userName = localStorage.getItem('user_name');
		isRememberMe = userName !== null;
		if(isRememberMe){
			username = userName;
		}
	});
	function handleRememberMe(){
		if(isRememberMe){
			localStorage.setItem("user_name", username);
		}
		else {
			localStorage.removeItem("user_name");
		}
	}
	async function onSubmit(e) {
		isSubmitting = true;
		handleRememberMe();
		e.preventDefault();
		await getToken(username, password, () => {
			isOpen = true;
			msg = $_('Authentication success');
			status = 'success';
			const redirectUrl = $page.url.searchParams.get('redirect');
			isSubmitting = false;
			resetLocalStorage();
			if (redirectUrl) {
				window.location.href = decodeURIComponent(redirectUrl);
			} else {
				goto('page/dashboard');
			}
		}, () => {
			isSubmitting = false;
			isOpen = true;
			status = 'danger';
			msg = $_('Incorrect user name or password.')
			setTimeout(() => {
				isOpen = false;
				status = '';
				msg = '';
			}, 3000);
		});
		isSubmitting = false;
	}

	function onPasswordToggle() {
		var x = document.getElementById('user-password');
		if (!x) return;

		if (x.type === 'password') {
			x.type = 'text';
			var icon = document.getElementById('password-eye-icon');
			icon.className = 'mdi mdi-eye-off-outline';
		} else {
			x.type = 'password';
			var icon = document.getElementById('password-eye-icon');
			icon.className = 'mdi mdi-eye-outline';
		}
	}
</script>

<Headtitle title="Login" />
<div class="account-pages my-5 pt-sm-5">
	<Container>
		<Row class="justify-content-center">
			<Col md={8} lg={6} xl={5}>
				<Card class="overflow-hidden">
					<div class="bg-primary-subtle">
						<Row>
							<Col class="col-7">
								<div class="text-primary p-4">
									<h5 class="text-primary">{$_('Welcome Back !')}</h5>
									<p>{$_('Sign in to continue to')} {PUBLIC_BRAND_NAME}.</p>
								</div>
							</Col>
							<Col class="col-5 align-self-end">
								<img src={PUBLIC_LOGIN_IMAGE} alt="" class="img-fluid" />
							</Col>
						</Row>
					</div>
					<CardBody class="pt-0">
						<div class="auth-logo">
							<Link href="page/dashboard" class="auth-logo-light">
								<div class="avatar-md profile-user-wid mb-4">
									<span class="avatar-title rounded-circle bg-light">
										<img src={PUBLIC_LOGO_URL} alt="" class="rounded-circle" height="55" />
									</span>
								</div>
							</Link>
							<Link href="page/dashboard" class="auth-logo-dark">
								<div class="avatar-md profile-user-wid mb-4">
									<span class="avatar-title rounded-circle bg-light">
										<img src={PUBLIC_LOGO_URL} alt="" class="rounded-circle" height="55" />
									</span>
								</div>
							</Link>
						</div>
						<div class="p-2">
							<Alert {isOpen} color={status}>{msg}</Alert>
							<Form class="form-horizontal" on:submit={onSubmit}>
								<div class="mb-3">
									<Label for="username" class="form-label">{$_('Username')}</Label>
									<Input
										type="text"
										class="form-control"
										id="username"
										placeholder={$_('Enter username')}
										disabled={isSubmitting}
										bind:value={username}
									/>
								</div>

								<div class="mb-3">
									<Label class="form-label" for="user-password">{$_('Password')}</Label>
									<div class="input-group auth-pass-inputgroup">
										<Input
											type="password"
											class="form-control"
											id="user-password"
											placeholder={$_('Enter password')}
											disabled={isSubmitting}
											aria-label="Password"
											aria-describedby="password-addon"
											bind:value={password}
										/>
										<Button
											color="light"
											type="button"
											id="password-addon"
											disabled={isSubmitting}
											on:click={() => onPasswordToggle()}
										>
											<i id="password-eye-icon" class="mdi mdi-eye-outline" />
										</Button>
									</div>
								</div>

								<div class="form-check">
									<input
										class="form-check-input"
										type="checkbox"
										id="remember-check"
										disabled={isSubmitting}
										bind:checked={isRememberMe}
									/>
									<Label class="form-check-label" for="remember-check">{$_('Remember me')}</Label>
								</div>

								<div class="mt-3 d-grid">
									<Button
										color="primary"
										disabled={isSubmitting}
										class="waves-effect waves-light"
										type="submit"
									>
										{!isSubmitting ? $_('Log In') : $_('Log In...')}
									</Button>
								</div>
								{#if PUBLIC_AUTH_ENABLE_SSO == 'true'}
								<div class="mt-4 text-center">
									<h5 class="font-size-14 mb-3">{$_('Sign in with')}</h5>

									<ul class="list-inline">
										<li class="list-inline-item">
											<Link
												class="social-list-item bg-primary text-white border-primary"
												href="{PUBLIC_SERVICE_URL}/sso/GitHub?redirectUrl={PUBLIC_LIVECHAT_HOST}page/user/me"
												disabled={isSubmitting}
											>
												<i class="mdi mdi-github" />
											</Link>
										</li>		
										<li class="list-inline-item">
											<Link
												class="social-list-item bg-primary text-white border-primary"
												href="{PUBLIC_SERVICE_URL}/sso/Keycloak?redirectUrl={PUBLIC_LIVECHAT_HOST}page/user/me"
												disabled={isSubmitting}
											>
												<i class="mdi mdi-cloud" />
											</Link>
										</li>									
										<li class="list-inline-item">
											<Link
												class="social-list-item bg-danger text-white border-danger"
												href="{PUBLIC_SERVICE_URL}/sso/Google?redirectUrl={PUBLIC_LIVECHAT_HOST}page/user/me"
												disabled={isSubmitting}
											>
												<i class="mdi mdi-google" />
											</Link>
										</li>
									</ul>
								</div>
								{/if}
								{#if PUBLIC_AUTH_ENABLE_FIND_PWD == 'true' }
								<div class="mt-4 text-center">
									<Link href="recoverpw" class="text-muted" disabled={isSubmitting}>
										<i class="mdi mdi-lock me-1" /> {$_('Forgot your password?')}
									</Link>
								</div>
								{/if}
							</Form>
						</div>
					</CardBody>
				</Card>
				<div class="mt-5 text-center">
					<p hidden={!(PUBLIC_ALLOW_SIGNUP === 'true')}>
						{$_("Don't have an account ?")}
						<Link href="register" class="fw-medium text-primary" disabled={isSubmitting}>{$_('Signup now')}</Link>
					</p>
					<p>
						© {new Date().getFullYear()}
						{PUBLIC_COMPANY_NAME}. {$_('Crafted with')}
						<i class="mdi mdi-heart text-danger" /> {$_('by open source community')}
					</p>
				</div>
			</Col>
		</Row>
	</Container>
</div>
