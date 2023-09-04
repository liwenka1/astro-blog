---
author: 李文凯
pubDatetime: 2023-09-04T20:15:24+08:00
title: 从零开始的Web端React音乐播放器（三）
postSlug: react-music-03
featured: true
draft: false
tags:
  - hook
ogImage: ""
description: 将登录功能完成并封装为 hook
canonicalURL: ""
---

## 前言

我们之前已经将项目配置完成，这一节我们将登录功能完成并封装为 hook

### 项目地址

前端地址：[react-music](https://github.com/liwenka1/react-music)
后端地址：[NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)

## 预览图

![verifyLogin.png](/assets/reactMusic/verifyLogin.png)
![qrLogin.png](/assets/reactMusic/qrLogin.png)

## 简单分析

- 布局不进行过多讲解，感兴趣的可以看下面的相关代码或者自行下载代码了解，重点讲解逻辑思路
- 首先，登录分为传统的验证码登录和二维码登录
- 验证码登录的要点在于，当点击获取验证码后需要对按钮进行 60s 的禁止点击
- 二维码的要点则是我们需要在点击弹框出来的时候利用一个定时器在一定时间内循环的去生成新的二维码，防止扫描时二维码过期
- 以及登录成功后关闭弹窗

## 代码实现

### 验证码登录

通过定义一个 countdown 来记录倒计时时间，以及在 useEffect 中启动定时器并在合适的时机进行清除，从而实现验证码的点击禁用操作，同时将电话以及验证码进行定义用于登录请求操作

```typescript
const [countdown, setCountdown] = useState(0);
useEffect(() => {
	let timer: NodeJS.Timeout;
	if (countdown > 0) {
		timer = setInterval(() => {
			setCountdown(countdown - 1);
		}, 1000);
	}
	return () => {
		clearInterval(timer);
	};
}, [countdown]);

const [phone, setPhone] = useState("");
const [verificationCode, setVerificationCode] = useState("");
const handleGetVerificationCode = async () => {
	const { data, message } = await useCaptchaSent(phone);
	if (data) {
		toast({
			title: "获取验证码",
			description: "已发送验证码到手机请查收",
		});
	} else {
		toast({
			title: "获取验证码",
			description: message,
		});
	}
	setCountdown(60);
};
const verifyLogin = async () => {
	const { data } = await useCaptchaVerify(phone, verificationCode);
	if (data) {
		try {
			const { code, cookie } = await useLoginCellphone(phone, verificationCode);
			if (code == 200) {
				const { profile } = await useLoginStatus(cookie);
				setIsLogin(true);
				setProfile(profile);
				localStorage.setItem("profile", JSON.stringify(profile));
				localStorage.setItem("cookie", cookie);
				toast({
					title: "登录",
					description: "授权登录成功",
				});
				setIsModalOpen(false);
			}
		} catch {
			toast({
				title: "登录",
				description: "登录出错，请尝试二维码登录",
			});
		}
	} else {
		toast({
			title: "登录",
			description: "验证码错误",
		});
	}
};
```

### 二维码登录

同理利用 useEffect 的性质来定义定时器并在合适的时机进行清除，同时定义 qrImg 用于存储二维码图片来进行相应的展示

```typescript
const [qrImg, setQrImg] = useState("");

useEffect(() => {
	let timer: NodeJS.Timeout;
	const qrLogin = async () => {
		const { unikey } = await useLoginQrKey();
		const { qrimg } = await useLoginQrCreate(unikey);
		setQrImg(qrimg);
		timer = setInterval(async () => {
			const { code, cookie } = await useLoginQrCheck(unikey);
			if (code === 800) {
				toast({
					title: "登录",
					description: "二维码已过期，请重新获取",
				});
				clearInterval(timer);
			}
			if (code === 803) {
				const { profile } = await useLoginStatus(cookie);
				setIsLogin(true);
				setProfile(profile);
				localStorage.setItem("profile", JSON.stringify(profile));
				localStorage.setItem("cookie", cookie);
				toast({
					title: "登录",
					description: "授权登录成功",
				});
				setIsModalOpen(false);
				clearInterval(timer);
			}
		}, 3000);
	};
	if (isModalOpen) qrLogin();

	return () => {
		clearInterval(timer);
	};
}, [isModalOpen]);
```

## 完整代码

### user.tsx

```tsx
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import {
	Dialog,
	DialogContent,
	DialogDescription,
	DialogHeader,
	DialogTitle,
	DialogTrigger,
} from "@/components/ui/dialog";
import { ModeToggle } from "../mode-toggle";
import Login from "./login";
import { Button } from "../ui/button";
import { useLogin } from "@/hooks/useLogin";

const User: React.FC = () => {
	const {
		isLogin,
		profile,
		handleLogoutClick,
		countdown,
		phone,
		verificationCode,
		qrImg,
		isModalOpen,
		handleGetVerificationCode,
		setPhone,
		setVerificationCode,
		setIsModalOpen,
		verifyLogin,
	} = useLogin();

	return (
		<>
			<div className="flex items-center space-x-3 pr-4 sm:space-x-6">
				{!isLogin && (
					<div className="flex items-center">
						<Avatar className="h-6 w-6 cursor-pointer rounded-full object-cover">
							<AvatarImage src="https://github.com/shadcn.png" />
							<AvatarFallback>CN</AvatarFallback>
						</Avatar>
						<Dialog open={isModalOpen} onOpenChange={setIsModalOpen}>
							<DialogTrigger
								className="ml-4 cursor-pointer text-xs font-light"
								onClick={() => setIsModalOpen(true)}
							>
								点我登录
							</DialogTrigger>
							<DialogContent>
								<DialogHeader>
									<DialogTitle>登录</DialogTitle>
								</DialogHeader>
								<Login
									countdown={countdown}
									phone={phone}
									verificationCode={verificationCode}
									qrImg={qrImg}
									handleGetVerificationCode={handleGetVerificationCode}
									setPhone={setPhone}
									setVerificationCode={setVerificationCode}
									setIsModalOpen={setIsModalOpen}
									verifyLogin={verifyLogin}
								/>
							</DialogContent>
						</Dialog>
					</div>
				)}
				{isLogin && (
					<div className="flex items-center">
						<Avatar className="h-6 w-6 cursor-pointer rounded-full object-cover">
							<AvatarImage src={profile.avatarUrl} />
							<AvatarFallback>CN</AvatarFallback>
						</Avatar>
						<Dialog>
							<DialogTrigger className="ml-4 cursor-pointer text-xs font-light">
								退出登录
							</DialogTrigger>
							<DialogContent>
								<DialogHeader>
									<DialogTitle>退出登录</DialogTitle>
								</DialogHeader>
								<DialogDescription>确定要退出登录吗?</DialogDescription>
								<DialogTrigger>
									<div className="mt-4 grid grid-cols-2 gap-4">
										<Button variant="secondary">取消</Button>
										<Button onClick={handleLogoutClick}>确定</Button>
									</div>
								</DialogTrigger>
							</DialogContent>
						</Dialog>
					</div>
				)}
				<ModeToggle />
			</div>
		</>
	);
};

export default User;
```

### login.tsx

```tsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Input } from "../ui/input";
import { Label } from "@radix-ui/react-dropdown-menu";
import { Button } from "../ui/button";

interface Props {
	countdown: number;
	phone: string;
	verificationCode: string;
	qrImg: string;
	handleGetVerificationCode: () => void;
	setPhone: React.Dispatch<React.SetStateAction<string>>;
	setVerificationCode: React.Dispatch<React.SetStateAction<string>>;
	setIsModalOpen: React.Dispatch<React.SetStateAction<boolean>>;
	verifyLogin: () => void;
}

const Login: React.FC<Props> = props => {
	const {
		countdown,
		phone,
		verificationCode,
		qrImg,
		handleGetVerificationCode,
		setPhone,
		setVerificationCode,
		verifyLogin,
	} = props;

	return (
		<>
			<div className="h-1/2">
				<Tabs defaultValue="captcha" className="w-full">
					<div className="flex justify-center">
						<TabsList>
							<TabsTrigger value="captcha">短信验证码登录</TabsTrigger>
							<TabsTrigger value="loginQr">二维码登录</TabsTrigger>
						</TabsList>
					</div>
					<TabsContent value="captcha" className="h-full">
						<div className="space-y-4">
							<Label>手机号</Label>
							<Input
								id="phone"
								value={phone}
								onChange={e => setPhone(e.target.value)}
							/>
						</div>
						<div className="mt-4 space-y-4">
							<Label>验证码</Label>
							<Input
								id="verificationCode"
								value={verificationCode}
								onChange={e => setVerificationCode(e.target.value)}
							/>
						</div>
						<div className="mt-4 grid grid-cols-2 gap-4">
							<Button
								variant="secondary"
								disabled={countdown > 0}
								onClick={handleGetVerificationCode}
							>
								{countdown > 0 ? `重新发送(${countdown}s)` : "获取验证码"}
							</Button>
							<Button onClick={verifyLogin}>登录</Button>
						</div>
					</TabsContent>
					<TabsContent value="loginQr" className="h-full">
						<div className="my-6 flex justify-center">
							<img src={qrImg} alt="二维码" />
						</div>
					</TabsContent>
				</Tabs>
			</div>
			<span className="text-xs font-light">
				为了安全起见目前仅支持短信验证码以及二维码进行登录，请放心的登录你的网易云账号
			</span>
		</>
	);
};
export default Login;
```

### useLogin.ts

```typescript
import { useEffect, useState } from "react";
import {
	useLoginCellphone,
	useLoginQrCheck,
	useLoginQrCreate,
	useLoginQrKey,
	useLoginStatus,
	useLogout,
} from "@/api/login";
import { useCaptchaSent, useCaptchaVerify } from "@/api/captcha";
import { UserProfile } from "@/api/login/type";
import { useToast } from "@/components/ui/use-toast";

export const useLogin = () => {
	const [isLogin, setIsLogin] = useState(false);
	const [profile, setProfile] = useState({} as UserProfile);
	const loginStatus = async () => {
		const cookie = localStorage.getItem("cookie");
		if (cookie) {
			const { profile } = await useLoginStatus(cookie);
			localStorage.setItem("profile", JSON.stringify(profile));
			setIsLogin(true);
			setProfile(profile);
		}
	};
	useEffect(() => {
		loginStatus();
	}, []);
	const handleLogoutClick = async () => {
		const { code } = await useLogout();
		if (code == 200) {
			localStorage.removeItem("cookie");
			localStorage.removeItem("profile");
			setIsLogin(false);
			setProfile({} as UserProfile);
		}
	};

	const [isModalOpen, setIsModalOpen] = useState(false);
	const { toast } = useToast();
	const [countdown, setCountdown] = useState(0);
	useEffect(() => {
		let timer: NodeJS.Timeout;
		if (countdown > 0) {
			timer = setInterval(() => {
				setCountdown(countdown - 1);
			}, 1000);
		}
		return () => {
			clearInterval(timer);
		};
	}, [countdown]);

	const [phone, setPhone] = useState("");
	const [verificationCode, setVerificationCode] = useState("");
	const handleGetVerificationCode = async () => {
		const { data, message } = await useCaptchaSent(phone);
		if (data) {
			toast({
				title: "获取验证码",
				description: "已发送验证码到手机请查收",
			});
		} else {
			toast({
				title: "获取验证码",
				description: message,
			});
		}
		setCountdown(60);
	};
	const verifyLogin = async () => {
		const { data } = await useCaptchaVerify(phone, verificationCode);
		if (data) {
			try {
				const { code, cookie } = await useLoginCellphone(
					phone,
					verificationCode
				);
				if (code == 200) {
					const { profile } = await useLoginStatus(cookie);
					setIsLogin(true);
					setProfile(profile);
					localStorage.setItem("profile", JSON.stringify(profile));
					localStorage.setItem("cookie", cookie);
					toast({
						title: "登录",
						description: "授权登录成功",
					});
					setIsModalOpen(false);
				}
			} catch {
				toast({
					title: "登录",
					description: "登录出错，请尝试二维码登录",
				});
			}
		} else {
			toast({
				title: "登录",
				description: "验证码错误",
			});
		}
	};

	const [qrImg, setQrImg] = useState("");

	useEffect(() => {
		let timer: NodeJS.Timeout;
		const qrLogin = async () => {
			const { unikey } = await useLoginQrKey();
			const { qrimg } = await useLoginQrCreate(unikey);
			setQrImg(qrimg);
			timer = setInterval(async () => {
				const { code, cookie } = await useLoginQrCheck(unikey);
				if (code === 800) {
					toast({
						title: "登录",
						description: "二维码已过期，请重新获取",
					});
					clearInterval(timer);
				}
				if (code === 803) {
					const { profile } = await useLoginStatus(cookie);
					setIsLogin(true);
					setProfile(profile);
					localStorage.setItem("profile", JSON.stringify(profile));
					localStorage.setItem("cookie", cookie);
					toast({
						title: "登录",
						description: "授权登录成功",
					});
					setIsModalOpen(false);
					clearInterval(timer);
				}
			}, 3000);
		};
		if (isModalOpen) qrLogin();

		return () => {
			clearInterval(timer);
		};
	}, [isModalOpen]);

	return {
		isLogin,
		profile,
		handleLogoutClick,
		countdown,
		phone,
		verificationCode,
		qrImg,
		isModalOpen,
		handleGetVerificationCode,
		setPhone,
		setVerificationCode,
		setIsModalOpen,
		verifyLogin,
	};
};
```
