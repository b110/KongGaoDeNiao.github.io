---
layout: post
title: pinyin4j汉字转拼音处理多音字的问题-重庆zhongqing
date: 2017-12-01
categories: blog
tags: [pinyin4j]
description: 2017-12-01-pinyin4j汉字转拼音处理多音字的问题-重庆zhongqing。
---

### pinyin4j汉字转拼音处理多音字的问题
#### 问题背景：
```
   查询列表中有时候需要按照中文首字母进行排序,这时候一般需要将字段转为拼音进行排序最为稳妥。
    例如：姓名按照汉字的拼音排序、省份按照拼音排序
```
#### 解决方案一：直接使用mysql语句进行排序（适合简单的不复杂的可以）
在MySQL数据库中使用UTF-8的编码进行排序会出现不按照中文拼音的顺序排序
解决这个问题的方案是把编码重新设定为***GBK或者GB2312***
但是问题又来了，数据库重设编码实在是个大问题，显然不能这样使用convert()这个函数可以实现临时编码并且解决问题
```
SQL语句：
正序： 
select * from table_name ORDER BY CONVERT(name USING gbk);
倒序：
select * from  table_name ORDER BY CONVERT(name USING gb2312) desc 
```
##### 注意：
![这里写图片描述](http://img.blog.csdn.net/20171201170047528?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjI5MDYzNDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 解决方案二：使用pinyin4j这个jar将汉字转为拼音，再进行compareTo 
##### 1.PinyinDuoYinziUtil工具类
```
package com.highset;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.HanyuPinyinVCharType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;

/**
 * <b>类描述：多音字排序工具类</b><br/>
 * <b>类名称：</b>PinyinDuoYinziUtil.java<br/>
 * <b>创建人：</b>jie.zhao<br/>
 * <b>创建时间：</b>2017年10月16日<br/>
 * <b>备注：</b><br/>
 *
 * @version 1.0.0<br/>
 */
public class PinyinDuoYinziUtil {

	private static Map<String, List<String>> pinyinMap = new HashMap<String, List<String>>();
	
	/**
	 * 静态代码块 -->加载字典文件 
	 */
	static {
		try {
			initPinyin("dict.txt");// 这是自定义字典文件
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
	}

	/**
	 * <b>Description:主程序</b><br> 
	 * @param args
	 * @throws FileNotFoundException
	 * @throws UnsupportedEncodingException
	 * @Note
	 * <b>Author:</b>Jie.Zhao</a>
	 * <br><b>Date:</b> 2017年12月1日 下午5:21:10
	 * <br><b>Version:</b> 1.0
	 */
	public static void main(String[] args) throws FileNotFoundException, UnsupportedEncodingException {
		List<String> list = new ArrayList<String>();
		list.add("浙江");
		list.add("北京");
		list.add("重庆");
		list.add("云南");
		list.add("和");
		list.add("天津");
		System.out.println("排序前："+list);
		Collections.sort(list, new Comparator<String>() {
			public int compare(String o1, String o2) {
				return getPinYin(o1).compareTo(getPinYin(o2));
			}
		});
		System.out.println("排序后："+list);

	}
		
	/**
	 * <b>Description: 这是核心方法</b><br> 
	 * @param chinese
	 * @return
	 * @Note
	 * <b>Author:</b>Jie.Zhao</a>
	 * <br><b>Date:</b> 2017年12月1日 下午5:22:05
	 * <br><b>Version:</b> 1.0
	 */
	public static String getPinYin(String chinese) {
		char[] arr = null;
		arr = chinese.toCharArray();
		String[] results = new String[arr.length];
		HanyuPinyinOutputFormat format = new HanyuPinyinOutputFormat();

		format.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		format.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		format.setVCharType(HanyuPinyinVCharType.WITH_V);
		StringBuffer pinyin = new StringBuffer();
		String result = "";
		try {
			for (int i = 0; i < arr.length; i++) {
				char ch = arr[i];
				// 判断是否为汉字字符
				if (java.lang.Character.toString(ch).matches("[\\u4E00-\\u9FA5]+")) {
					results = net.sourceforge.pinyin4j.PinyinHelper.toHanyuPinyinStringArray(ch, format);
					int len = results.length;

					if (len == 1) { // 不是多音字
						result = results[0];
					} else if (results[0].equals(results[1])) {// 非多音字 有多个音，取第一个
						result = results[0];
					} else {
						// 多音字
						int length = chinese.length();
						boolean flag = false;
						String s = null;
						List<String> keyList = null;
						for (int x = 0; x < len; x++) {
							String py = results[x];
							if (py.contains("u:")) { // 过滤 u:
								py = py.replace("u:", "v");
								// System.out.println("filter u:" + py);
							}
							keyList = pinyinMap.get(py);
							if (i + 3 <= length) { // 后向匹配2个汉字 大西洋
								s = chinese.substring(i, i + 3);
								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("last 2 > " + py);
									result = py;
									flag = true;
									break;
								}
							}

							if (i + 2 <= length) { // 后向匹配 1个汉字 大西
								s = chinese.substring(i, i + 2);
								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("last 1 > " + py);
									result = py;
									flag = true;
									break;
								}
							}

							if ((i - 2 >= 0) && (i + 1 <= length)) { // 前向匹配2个汉字 龙固大
								s = chinese.substring(i - 2, i + 1);
								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("before 2 < " + py);
									result = py;
									flag = true;
									break;
								}
							}

							if ((i - 1 >= 0) && (i + 1 <= length)) { // 前向匹配1个汉字 固大
								s = chinese.substring(i - 1, i + 1);
								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("before 1 < " + py);
									result = py;
									flag = true;
									break;
								}
							}

							if ((i - 1 >= 0) && (i + 2 <= length)) { // 前向1个，后向1个 固大西
								s = chinese.substring(i - 1, i + 2);
								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("before last 1 <> " + py);
									result = py;
									flag = true;
									break;
								}
							}
						}

						if (!flag) { // 都没有找到，匹配默认的 读音 大
							s = String.valueOf(ch);
							for (int x = 0; x < len; x++) {
								String py = results[x];
								if (py.contains("u:")) { // 过滤 u:
									py = py.replace("u:", "v");
									// System.out.println("filter u:");
								}
								keyList = pinyinMap.get(py);

								if (keyList != null && (keyList.contains(s))) {
									// System.out.println("default = " + py);
									result = py;// 拼音首字母 大写
									break;
								}
							}
						}
					}
					pinyin.append(result);
				} else {
					pinyin.append(java.lang.Character.toString(arr[i]));

				}
			}
			return pinyin.toString();
		} catch (BadHanyuPinyinOutputFormatCombination e1) {
			e1.printStackTrace();
		}
		// System.out.println(pinyin.toString());
		return pinyin.toString();
	}

	/**
	 * <b>Description:初始化 所有的多音字词组</b><br> 
	 * @param filePath
	 * @throws FileNotFoundException
	 * @throws UnsupportedEncodingException
	 * @Note
	 * <b>Author:</b>Jie.Zhao</a>
	 * <br><b>Date:</b> 2017年12月1日 下午5:22:22
	 * <br><b>Version:</b> 1.0
	 */
	public static void initPinyin(String filePath) throws FileNotFoundException, UnsupportedEncodingException {
		// 读取多音字的全部拼音表;
		InputStreamReader isr = new InputStreamReader(new FileInputStream(filePath), "gbk");
		BufferedReader br = new BufferedReader(isr);
		String s = null;
		try {
			while ((s = br.readLine()) != null) {
				if (s != null) {
					String[] arr = s.split("#");
					String pinyin = arr[0];
					String chinese = arr[1];
					if (chinese != null) {
						String[] strs = chinese.split(" ");
						List<String> list = Arrays.asList(strs);
						pinyinMap.put(pinyin, list);
					}
				}
			}
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				br.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

##### 2.新建一个duoyinzi_dict.txt(字典文件，可按照需求自己定义)，文件内容如下

```
a#阿 阿姨 阿富 阿门 阿拉 阿林  黑阿  麦阿密 鹿城阿岙 阿福  
ao#拗口 违拗 凹  
ai#艾 艾滋 艾蒿 未艾  
bang#膀 翅膀 臂膀 重磅 磅秤 黄泥磅店 蛤蚌 蚌壳 河蚌 鹬蚌 珠蚌 蚌  
ba#扒  
bai#叔伯 百 百万  柏  
bao#剥皮 薄 超薄 薄脆 薄板 薄饼  暴 暴晒 暴发 暴雨 暴力 风暴  暴露 暴风  汉堡 古堡 地堡 城堡 龍堡 卡斯堡  麻家堡 麦芬堡 汉堡 麦得堡  麦尔堡  曝光 瀑河  
beng#蚌埠  
bi#复辟  臂 臂章 螳臂 交臂 前臂 一臂 奋臂 膀臂 臂膀 秘鲁 泌阳  
bing#屏弃 屏气 屏除 屏退 屏息  
bian#扁 扁桃 便 方便 方便面 便当 便捷  
bo#薄 薄荷 单薄 伯 伯仲 伯乐 伯劳 伯父 大伯 老伯 伯母 黄伯 伯爵 停泊 淡泊 尼泊 漂泊 波 鸿波 柏林  
bu#大埔  
can#参 参谋 参事 总参 参数 参议 参观 参拜 参股  
cang#藏 埋藏 藏头 秘藏  雪藏 藏匿 收藏 馆藏 矿藏 隐藏  蕴藏 藏袍 储藏 窖藏 藏龙  藏胞 冷藏 珍藏 私藏 藏掖 西藏 藏书 藏品 伧俗  伧 龙藏寺  
cen#参差  
ceng#曾 不曾 似曾 几曾 何曾 曾经 曾几 未曾 噌  噌的 一声  
cha#差  刹那 宝刹 一刹  喳喳  
chai#公差 差役 专差 官差 听差 美差 办差 差事 差使 肥差 当差 钦差  
chan#颤 颤悠  单于 禅 禅学 班禅 禅宗 禅堂 禅门 禅机 禅杖 禅房 禅师 坐禅 参禅 禅院  
chang#长 周长 细长 长发 三长 长河 长袖 长衫 天长 长短 超长 长沙 长春 长远 长度 长江 长处 长假  长街 长征 全长 长城 波长  身长 长途 长吁 长虹 长方  厂  
chao#朝 朝阳 朝阳区 朝鲜 朝廷 王朝 历朝 解嘲 讥嘲 自嘲 嘲笑 嘲弄 冷嘲 嘲讽 绰绰 绰起 绰家 剿袭 剿说  
che#车 汽车 停车场 车车  黑车 车饰  
chen#称职 匀称 称心 相称 对称  
cheng#称 职称 简称 总称 官称 代称 称号 称谓  昵称 谦称 全称 名称  乘 噌吰 澄  
chu#六畜 家畜 耕畜 畜生 牲畜  
chui#椎心  
chuan#传 文传 传媒 传销 传情 真传 祖传 传闻 传家 秘传 传单 传说  
chi#匙子 茶匙 羹匙 汤匙 尺 尺度 英尺 咫尺 尺码 公尺 卡尺 米尺 卷尺  
chong#重庆 重重  
chou#臭 汗臭 臭氧 口臭 腋臭 臭虫 臭骂 臭美 酸臭 腐臭 臭气 腥臭 臭名 遗臭 恶臭 臭豆 狐臭 臭味 臭架  
chuang#经幢  
chuo#绰 绰约 阔绰 绰号 宽绰  
ci#参差 伺候 龟兹  
cuan#攒钱 攒聚 攒动   
cuo#撮儿 撮要 撮合  
da#大 大街 沓子 龙大 大西洋 大昌  大圣 福大 黑大 大华 大包  大厦  
dao#叨 叨唠 絮叨 叨念 叨咕 念叨 唠叨 叨叨 磨叨  
dai#大夫  
dan#单 西单 东单 清单 报单 单利 名单 单姓 单亲 单线 单科 单间 单挑 单价 单词  子弹    
de#的 似的 总的 中的 别的  
deng#澄清  
di#怎的 无的 有的 目的 标的 打的 的确 的当 的士 上地 大地 天地 提防 堤  
diao#调 蓝调 蓝调吧 调调 音调 论调 格调 调令 低调 笔调 基调 强调 声调 滥调 老调 色调 单调 腔调 跑调 曲调 步调 语调 主调 情调  
du#都 都会 国都 都城 古都 故都 大都 首都 成都 旧都 都市 龙都  鼎都 鹤都 鹏都 鸿都  麦度 度 态度 读书 法度 宽度 进度   
dui#堆    
dou#全都 句读  
duo#测度 忖度 揣度 猜度 舵  
dun#粮囤 顿  
e#阿谀 阿胶 阿弥 恶心  
er#儿  
fan#番 番茄 繁  
fo#佛 佛塔 佛徒 佛牙 佛教  
fu#仿佛 果脯  
fou#否 是否 与否  
ga#咖 咖喱 伽马  
gai#盖  
gang#扛鼎  
ge#革 革命  皮革  鹰革  蛤蚧 文蛤 蛤蜊 咯吱 咯噔 咯咯  
gei#给  
geng#脖颈  
gong#女红  
gu#谷 布谷 谷物 谷地 硅谷 中鹄  麦谷 麓谷 鹭谷 鼓  
gui#龟 龟山 龟士 龟博 龟仔 鹿龟  龟汁 龟苓 龟顶  
gua#挺括 顶呱 呱呱 呱唧 呱嗒 呱  
guan#纶巾 东莞  
guang#广 广州 广东 广播  
ha#蛤蟆 癞蛤 虾蟆  
hai#还是 还有 咳  
hao#貉子 貉绒  
hang#总行 分行 支行 行业 排行 行情 央行 商行 外行 银行 商行 酒行 麻行 琴行 巷道 珩  
he#和 嘉和 和睦 亲和  龙和  之貉 威吓 恫吓 恐吓 鼎和  锦和 麒和苑 合 合资 鸿合  
heng#道行  
hu#鹄 鹄望 鸿鹄 鹄立  
huan#还 鹂还  
hui#会 会馆 会展 会所 协会 国会 会堂  
hong#红 红装 红牌  红木 红人 虹  
huo#软和 热和 暖和  
ji#病革 给养 自给 给水 薪给 给予 供给 稽考 稽查 稽核 滑稽 稽留 缉获 缉查 缉私 缉捕 狼藉 奇数 亟  亟待 亟须 亟亟 亟需 诘屈 荠菜  
jia#雪茄 伽  瑜伽 伽利略 家  
jian#见 龙见  
jiang#降 降温 降低 降旗 下降 倔强  
jiao#嚼舌 嚼子 细嚼 角 平角 视角 海角 龙角 鹿角  围剿 征剿 饺 饺子 脚  
jie#解 解放 慰藉 蕴藉 盘诘 诘难 诘问 反诘 桔  
jin#矜 矜夸 矜持 骄矜 自矜 劲  
jing#颈 颈项 颈椎 引颈 长颈 宫颈 瓶颈 龙颈  黑颈鹤 鹿颈  景 景色 帝景 劲松  
ju#咀 咀嚼  居  桔汁  
jun#均 平均 鸿均  
juan#棚圈 圈养  
jv#咀嚼 趑趄  
jvan#猪圈 羊圈  
jue#主角 角色 旦角 女角 丑角 角力 名角 配角 咀嚼 觉 直觉 感觉 错觉 触觉 幻觉 堀  
jun#龟裂 俊  
jvn#龟裂  
ka#咖啡 卡 磁卡  贺卡 卡拉 胸卡 声卡 卡片  绿卡 卡通  网卡 卡口  龙卡  咯痰 咯血 喀  
kang#扛  
ke#咳 咳嗽 干咳 贝壳 蚌壳 外壳 蛋壳 脑壳 弹壳  
keng#吭声 吭气 吭哧  
kuai#会计 财会  
kuo#括  
la#癞痢 腊  
lai#癞疮 癞子 癞蛤 癞皮  
lao#积潦 络子 落枕 落价 麻粩  
le#乐 娱乐 玩乐 乐趣  美乐 乐缘  勒  了  
lei#勒紧  
lo#然咯  
lou#佝偻  
long#里弄 弄堂 泷  
li#礼 豊 栎  
liao#了解 了结  明了 了得 末了 未了 了如  了如指掌 潦草 潦倒  
liang#靓  
liu#碌碡  碌碌 劳碌 忙碌 庸碌 六  
lu#绿林 碌  
luo#络 络腮 落 部落 落花 日落  
lv#率 频率 机率 比率 效率 胜率 概率 汇率 功率 倍率 绿 绿叶 淡绿 绿色 绿豆 伛偻  绿洲  
lun#丙纶 锦纶 经纶 涤纶  
mai#埋  
man#埋怨 蔓  
mai#脉 山脉 动脉 命脉 筋脉 脉象 气脉 脉动 脉息 脉络 一脉 经脉  
mang#氓 流氓  
me#黛么  
meng#群氓 盟  
mei#没  
mo#埋没 隐没 脉脉 模 航模 模糊 男模 楷模 规模 劳模 模型 模范 模特 名模  摩 么 麼 麽  
mou#绸缪  
mi#秘 秘密 秘方 奥秘 神秘 泌尿 分泌  
miu#谬 谬论 纰缪  
mu#人模 字模 模板 模样 模具 装模 装模做样 模子  
na#哪 娜 安娜 娜娜 丽娜 黛尔娜 黛娜  海娜 黑娜 黄丽娜 麦香娜  优娜 麦娜 麟娜  那  
nan#南 南方 湖南  
ne#哪吒 呢  
nong#弄  
ni#毛呢 花呢 呢绒 线呢 呢料 呢子 呢喃 溺  
niao#便溺 尿  
nian#粘  
niu#执拗 拗不  
nue#疟 疟疾  
nuo#婀娜 袅娜  
nv#女 女人  
nve#疟原 疟蚊  
pa#扒  
pai#迫击 迫击炮 派  
pao#刨 炮  
pang#膀胱 膀肿 磅礴  
pi#否极 臧否 龙陂 黄陂  
pian#扁舟 便宜  
piao#朴姓  
ping#屏  屏幕 荧屏 银屏  
po#泊 迫 朴刀 坡 陂  
pu#暴十 一曝十寒 里堡 十里堡 胸脯 肉脯 脯子 杏脯 简朴 朴质 古朴 朴厚 纯朴 朴素 诚朴 俭朴 朴实 淳朴 曝晒 瀑布 飞瀑 埔 黄埔  
qiu#龟兹  
qi#稽首 缉鞋 栖 奇 奇妙 传奇 亟来 荸荠 蹊跷  林栖  鹿奇 鹭奇 漆 齐 齐天大圣 齐天 其  
qia#卡脖 卡子 关卡 卡壳 哨卡 边卡 发卡  
qiao#雀盲 雀子 地壳 甲壳 躯壳  
qian#纤手 拉纤 纤夫 纤绳 乾  
qiang#强颜  强人 自强 强烈 强风 强大 黎强 麒强 鹤强 龚强  
qie#茄子 颠茄 番茄 趔趄   
qin#亲 亲和 亲亲 棘矜 矜锄  
qing#干亲 亲家 黥  
qu#区 小区  
quan#转圈 钢圈 圆圈 罗圈 弧圈 垫圈 小圈 眼圈  
que#雀 麻雀 鸟雀 燕雀 孔雀 云雀 雀巢、  
re#般若  
ruo#若  
sai#塞 麦迪塞姆 活塞  
se#堵塞 搪塞 茅塞 闭塞 鼻塞 梗塞 阻塞 淤塞 拥塞 哽塞 月色 彩色 特色 深色 声色 黛色  黛色 黑色瞳 色坊  
sha#刹车 急刹 急刹车 厦  广厦 大厦 商厦 鹰大厦  莎  
shai#色子  
shan#姓单 单县  杉 铁杉 杉树 封禅 禅让 黒杉  栅  
shang#裳 衣裳  
she#拾级 折本  
shen#沙参 野参 参王 人参 红参 丹参 山参 海参 刺参 没什 什么 为什 鹿参 身  
sheng#野乘 千乘 史乘  省 晟 盛 盛大 鸿盛  
shi#钥匙 拾荒 捡拾 拾物 家什 什物 什锦 麻什  麦什 喀什 牛什  识  见识 知识 似的 骨殖 食 饮食 副食  石 石业 石头 石艺 氏 姓氏 上栅 下栅  
shuai#表率 率性 率直 率真 粗率 率领 轻率 直率 草率 大率 坦率 数字 招数 基数 数码  
shuang#泷水  
shu#属 金属 气数 岁数 度数 数据 级数 数控 数学 参数 次数 正数 代数 实数 系数 分数 辈数  
shui#游说  
shuo#数见 数见不鲜 传说 听说 妄说 实说  胡说 评说 分说 小说  
si#窥伺 伺弄 伺机 疑似 似是 好似 似曾 形似 酷似 貌似 似懂 胜似 恰似 近似 神似 赛似 看似 活似 强似 似乎 类似 相似 思  
su#宿主 宿命 归宿 住宿 借宿 寄宿 宿营 夜宿 露宿 投宿 宿舍 名宿 整宿 食宿  
sui#尿泡  
ta#拓本 拓片 碑拓 疲沓 拖沓 杂沓 沓 塔 鸿塔  
tang#汤 鸭汤 鸡汤  
tao#叨扰 叨光 陶 陶器  
tan#弹性 弹力 反弹  
ti#手提 提速 提意 提前 提早 提升 提议 提款 提婚 提包 耳提 提供 麦麦提 体  
tiao#空调 调教 烹调 调羹 调料 调皮  调控 调节 调整 调价 谐调  协调 调色 调侃 调味 失调 调治 调频 调剂 调停 调休 调解  
ting#町 域町 听  
tong#垌  
tui#褪色 褪毛  
tuo#拓 拓宽 拓荒 开拓 落拓 拓展 拓印  
tun#屯 囤积 囤聚  
wei#尾 响尾 尾巴 尾灯 船尾 追尾 尾椎 月尾  燕尾 尾数 年尾 岁尾 鸢尾 凤尾 彗尾 尾翼 结尾 遗之 龙尾  齐鑫尾 麻尾 麦度 鹿尾  
wu#可恶 交恶 好恶 厌恶 憎恶 嫌恶 痛恶 深恶  
wan#藤蔓 枝蔓 瓜蔓 蔓儿  莞尔 万 百万 萬  
xia#虾 虾仁 青虾 大虾 虾皮 对虾 虾子 虾酱 虾兵 虾米 龙虾 噶厦 厦门 吓唬 吓人 惊吓 天虾 龙虾 皮皮虾 麦虾  
xi#栖栖 系 关系 星系 水系 系念 体系 联系 系列 菜系 世系  蹊 蹊径 溪 洗  
xiao#校 学校 切削 削面 刀削 刮削  
xian#纤细 光纤 纤巧 纤柔 纤小 纤维 纤瘦 纤纤 化纤 纤秀 棉纤 纤尘  
xiang#巷 街巷 僻巷 巷子 龙门巷 六巷 龙湾巷 龙港巷 龙泉巷 龙巷 龙妙巷 龄巷 齐家巷 鼓楼巷 鼓巷 黎明巷 麻子巷 麻园巷 麦子巷 鹊巷  
xie#解数 出血 采血 换血 血糊 尿血 淤血 放血 血晕 血淋 便血 吐血 咯血 叶韵 蝎 蝎子  
xiu#铜臭 乳臭 成宿 星宿  
xin#馨 信 鸿信  
xing#深省 省视 内省 不省人事 省悟 省察 行 旅行 例行 行程 行乐 龙行 人行 流行 先行 行星 品行  发行 行政 风行 龙行 龍行 麟行  
xu#牧畜 畜产 畜牧 畜养 吁 气吁 喘吁 吁吁 麦埂圩  
xue#削 削减 削弱  削瘦 削球 削平 削价 瘦削 剥削 削职  删削 削肩  血 吸血  
xun#荨 荨麻 荨麻疹  
ya#芽  
yao#发疟 疟子 约斤 称约 钥匙 金钥 耀  
yan#吞咽 咽气 咽喉  殷红 腌 腌制 腌肉 腌菜 烟 烟草 名烟 烟酒  
ye#抽咽 哽咽 咽炎 下咽 呜咽 幽咽 悲咽 叶 绿叶 叶子 荷叶 落叶 菜叶 红叶 树叶 枫叶  茶叶 葉 鸿葉  液   
yi#自艾 惩艾 后尾 遗  屹  
yin#殷 殷勤 殷墟 殷切 殷鉴  
yo#杭育  
yu#谷浑 呼吁 吁请 吁求 育 体育 教育 育儿 熨帖 熨烫 於  
yuan#员  
yun#熨 熨斗 电熨斗  
yue#乐音 器乐 乐律 乐章 音乐 乐理 民乐 乐队 声乐 奏乐 弦乐 乐坛 管乐 配乐 乐曲 乐谱  锁钥 密钥 乐团 鼓乐社 乐器 栎阳 约 约会  
zan#积攒  
zang#宝藏 藏历 藏文 藏香 藏语 藏青 藏族 藏医 藏戏 藏药 藏蓝 蔵  
ze#择 择善  
zeng#曾孙 曾祖  
za#绑扎 结扎 包扎 捆扎  
zai#牛仔 龟仔 龙仔 鼻仔 羊仔  仔仔 麻仔  麵包仔 麦旺仔 鸿仔 煲仔 福仔  
zha#扎 马扎 挣扎 扎啤 扎根 扎手 扎针 扎花 扎堆 扎营 扎实 稳扎 柞水 麻扎镇 麻扎乡 喳 栅栏  
zhai#择菜  
zhan#不粘 粘贴 粘连  
zhao#朝朝 明朝 朝晖 朝夕 朝思 有朝 今朝 朝气 朝三 朝秦 朝霞 鹰爪 龙爪 魔爪 爪牙 失着 着数 龙爪槐  
zhe#折 破折 打折 叠 曲折 折冲 存折  折合 折旧 折纸 骨折 折返 折价 折算 波折 折扇 对折 不折 折扣 七折 折中 拙著 要著 著文 新著 着 本着 对着  
zhi#标识 嘎吱 咯吱 吱扭 吱吱 繁殖 增殖 养 生殖 殖民 枝  
zhong#重 重量 鹏重 种  
zhou#粥  
zhu#属意 著录 撰著 名著 专著 著述 著作 显著 昭著 原著 著名 著书 遗著 论著 著者 编著 卓著 译著 著称  
zhua#爪  
zhui#椎 椎骨 尾椎 椎间 腰椎 胸椎 颈椎 脊椎  
zhuo#执著 着装 着落 着意 着力 附着 着笔 胶着 着手 着重 穿着 衣着 执着 着眼 着墨 着实 沉着 着陆 着想 着色  
zhuang#幢房 一幢 幢楼  
zi#吱声 兹 来兹 今兹 仔细 仔猪  
zu#足 沐足 足道  
zuo#撮毛 小撮 柞绸 柞蚕 柞树 柞木  
zui#咀唇 尖沙咀 黄达咀 黄土咀 鹰咀
```

##### 运行结果，完美解决。。。。。
![这里写图片描述](http://img.blog.csdn.net/20171201172304696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjI5MDYzNDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


demo下载地址：
链接：https://pan.baidu.com/s/1o8LwXPw 密码：tbt0
链接失效的话请联系本人。。。。


github:https://github.com/KongGaoDeNiao/PinyinDuoYinzi

参考网址：
http://blog.csdn.net/hao_kkkkk/article/details/51780719
https://zhidao.baidu.com/question/488199891043049572.html













