<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import { gsap } from 'gsap';

const root = ref(null);
let ctx;
let revealObserver;

const navItems = [
  { label: '首页', href: '#hero' },
  { label: '项目', href: '#projects' },
  { label: '能力', href: '#skills' },
  { label: '时间线', href: '#timeline' },
  { label: '工作之外', href: '#outside' },
  { label: '联系', href: '#contact' },
];

const heroMetrics = [
  { label: 'PROJECTS', value: '03' },
  { label: 'RANK', value: 'TOP 5%' },
  { label: 'FOCUS', value: 'RAFT · SOLIDITY · VUE' },
  { label: 'LOCATION', value: 'CHONGQING' },
];

const projects = [
  {
    level: 'PRIMARY RECORD',
    code: '01',
    name: 'MedTrust-Raft',
    title: '医疗数据联盟治理平台',
    summary: '面向多医疗机构可信数据共享场景的联盟治理平台。',
    detail: '基于 Raft 共识机制构建核心协作逻辑。',
    role: '主导系统设计，并独立完成 Raft 核心机制实现。',
    points: [
      '从零实现 Raft 核心流程，覆盖 Leader 选举、日志复制与心跳维持等关键机制。',
      '围绕多机构协作场景设计联盟治理架构，提升医疗数据共享过程中的可信性与一致性。',
      '完成从共识层到服务层的整体工程串联，使系统具备可运行、可演示的完整形态。',
    ],
    tech: ['Go', 'Raft', 'Gin', 'BoltDB', 'Vue 3'],
  },
  {
    level: 'SUPPORT RECORD',
    code: '02',
    name: '公益众筹智能合约',
    summary: '基于以太坊的去中心化众筹合约实践。',
    role: '负责合约逻辑设计与关键安全机制实现。',
    points: [
      '围绕众筹流程实现目标验证、自动拨款与失败退款等核心逻辑。',
      '按里程碑拆分资金释放流程，增强资金使用的可控性与透明度。',
      '在合约设计中引入常见安全模式，降低重入等链上风险。',
    ],
    tech: ['Solidity', 'Ethereum', 'Truffle', 'Ganache'],
  },
  {
    level: 'SUPPORT RECORD',
    code: '03',
    name: 'AI 工具与技术文档实践',
    summary: '将 AI 工具、需求分析与技术文档能力结合到开发流程中的持续实践。',
    role: '负责方案整理、技术表达与辅助研发流程优化。',
    points: [
      '能够将复杂需求快速拆解为结构化方案，提高沟通与执行效率。',
      '在项目过程中持续输出需求分析、技术方案与架构说明文档。',
      '熟悉借助 AI 工具辅助信息整理、内容生成与开发支持，提升个人交付效率。',
    ],
    tech: ['AI Tools', 'Technical Writing', 'Solution Design'],
  },
];

const capabilityPillars = [
  {
    title: '分布式系统',
    text: '围绕一致性机制、服务协作与系统设计建立技术主轴。',
    keywords: ['Raft', '一致性机制', '服务协作', '系统设计'],
  },
  {
    title: '区块链开发',
    text: '理解链上执行逻辑，能够完成智能合约设计与测试实践。',
    keywords: ['Solidity', 'Ethereum', '智能合约', '链上逻辑'],
  },
  {
    title: '工程实现',
    text: '关注从方案、接口、数据存储到前端表达的完整落地过程。',
    keywords: ['Go', 'Vue 3', 'Gin', 'BoltDB'],
  },
];

const skillGroups = [
  { label: 'Languages', items: ['Go', 'Solidity', 'Python', 'JavaScript'] },
  { label: 'Backend & Infra', items: ['Gin', 'BoltDB', '共识机制实现', '服务架构'] },
  { label: 'Frontend', items: ['Vue 3', '交互实现', '页面工程化'] },
  { label: 'Blockchain', items: ['Ethereum', 'Truffle', 'Ganache', '智能合约开发'] },
];

const timeline = [
  {
    tag: 'STAGE 01',
    title: '专业起点',
    text: '进入区块链工程相关学习，建立对分布式系统与链上开发的基础认知。',
  },
  {
    tag: 'STAGE 02',
    title: '算法与竞赛训练',
    text: '通过比赛和系统性训练补强问题拆解、逻辑推演和实现能力。',
  },
  {
    tag: 'STAGE 03',
    title: '项目实践阶段',
    text: '开始把学习内容落到真实项目中，重点进入联盟治理、共识机制、智能合约等更成体系的实践。',
  },
  {
    tag: 'STAGE 04',
    title: '工程化表达与能力收束',
    text: '不再只是做功能，而是开始关注架构、方案、文档、协作和交付质量，形成更完整的工程思维。',
  },
];

const outsideItems = [
  {
    label: '观察与视觉表达',
    title: '绘画',
    text: '通过线条、结构与色彩训练观察力，也让自己在技术工作之外保留视觉表达与创造性的出口。',
  },
  {
    label: '节奏感与长期练习',
    title: '吉他',
    text: '把音乐当作调节节奏与保持专注的方式，长期的练习过程也影响了我对结构、节奏与耐心的理解。',
  },
  {
    label: '表达、协作与复盘',
    title: '文档与写作',
    text: '习惯把需求、方案与技术决策记录清楚，让复杂问题更容易被沟通、协作与复盘。',
  },
];

const contacts = [
  { label: 'EMAIL', value: '2489555099@qq.com', href: 'mailto:2489555099@qq.com' },
  { label: 'PHONE', value: '17883563946', href: 'tel:17883563946' },
  { label: 'GITHUB', value: 'github.com/1dryiwu1', href: 'https://github.com/1dryiwu1' },
];

function setupRevealAnimations() {
  const revealItems = root.value.querySelectorAll('.reveal');

  revealObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (!entry.isIntersecting) return;

        gsap.to(entry.target, {
          autoAlpha: 1,
          y: 0,
          duration: 0.75,
          ease: 'power3.out',
        });
        revealObserver.unobserve(entry.target);
      });
    },
    { threshold: 0.18, rootMargin: '0px 0px -8% 0px' },
  );

  revealItems.forEach((item) => revealObserver.observe(item));
}

onMounted(() => {
  if (!root.value) return;

  ctx = gsap.context(() => {
    const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    gsap.set('.reveal', { autoAlpha: 0, y: 28 });

    gsap.timeline({ defaults: { ease: 'power3.out' } })
      .from('.site-nav', { autoAlpha: 0, y: -16, duration: 0.6 })
      .from('.hero-kicker, .hero-title, .hero-name, .hero-copy, .hero-actions', {
        autoAlpha: 0,
        y: 24,
        duration: 0.7,
        stagger: 0.08,
      }, '-=0.2')
      .from('.metric-cell', {
        autoAlpha: 0,
        y: 14,
        duration: 0.5,
        stagger: 0.06,
      }, '-=0.15')
      .from('.core-shell', {
        autoAlpha: 0,
        scale: 0.94,
        duration: 0.8,
      }, '-=0.55');

    if (!reduceMotion) {
      gsap.to('.core-glass', {
        '--core-glow': 0.95,
        duration: 2.8,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
      });

      gsap.to('.stabilizer', {
        y: (index) => (index % 2 === 0 ? -8 : 8),
        duration: 3.8,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: 0.3,
      });

      gsap.to('.scan-blade', {
        yPercent: 230,
        autoAlpha: 0.7,
        duration: 3.2,
        repeat: -1,
        repeatDelay: 2.2,
        ease: 'power1.inOut',
      });

      gsap.to('.signal-dot', {
        x: 220,
        autoAlpha: 1,
        duration: 2.1,
        repeat: -1,
        repeatDelay: 2.8,
        ease: 'power2.inOut',
      });
    }

    setupRevealAnimations();
  }, root.value);
});

onUnmounted(() => {
  revealObserver?.disconnect();
  ctx?.revert();
});
</script>

<template>
  <main ref="root" class="portfolio-shell">
    <div class="page-noise"></div>
    <nav class="site-nav" aria-label="页面导航">
      <a class="brand-mark" href="#hero" aria-label="返回首页">YIWU</a>
      <div class="nav-links">
        <a v-for="item in navItems" :key="item.href" :href="item.href">{{ item.label }}</a>
      </div>
    </nav>

    <section id="hero" class="hero-section">
      <div class="hero-layout">
        <div class="hero-content">
          <p class="hero-kicker">PERSONAL DOSSIER / BLOCKCHAIN ENGINEERING</p>
          <h1 class="hero-title">易武</h1>
          <p class="hero-name">YI WU</p>
          <p class="hero-role">区块链开发者 / 分布式系统实践 / 智能合约工程</p>
          <p class="hero-copy">专注于将链上逻辑、系统设计与工程实现落到可运行的项目中。</p>
          <div class="hero-actions" aria-label="主要操作">
            <a href="#projects" class="primary-action">查看项目</a>
            <a href="mailto:2489555099@qq.com" class="secondary-action">联系我</a>
          </div>
        </div>

        <div class="core-visual" aria-label="纵向金属核心舱视觉">
          <div class="core-frame">
            <div class="stabilizer stabilizer-left"></div>
            <div class="stabilizer stabilizer-right"></div>
            <div class="core-shell">
              <div class="core-cap core-cap-top"></div>
              <div class="core-body">
                <div class="panel-line panel-line-a"></div>
                <div class="panel-line panel-line-b"></div>
                <div class="core-glass">
                  <span class="scan-blade"></span>
                  <span class="energy-column"></span>
                </div>
                <span class="lock-node node-top"></span>
                <span class="lock-node node-bottom"></span>
              </div>
              <div class="core-cap core-cap-bottom"></div>
            </div>
            <div class="signal-rail">
              <span class="signal-dot"></span>
            </div>
            <div class="system-label">CORE CHAMBER / STANDBY RUNNING</div>
          </div>
        </div>
      </div>

      <div class="metric-strip" aria-label="个人参数">
        <div v-for="metric in heroMetrics" :key="metric.label" class="metric-cell">
          <span>{{ metric.label }}</span>
          <strong>{{ metric.value }}</strong>
        </div>
      </div>
    </section>

    <section id="projects" class="content-section project-section">
      <div class="section-heading reveal">
        <p class="section-index">02 / SELECTED RECORDS</p>
        <h2>核心项目经历</h2>
        <p>围绕分布式系统、智能合约与工程实现展开的代表性项目。</p>
      </div>

      <div class="project-grid">
        <article class="project-panel primary-project reveal">
          <div class="record-head">
            <span>{{ projects[0].level }}</span>
            <strong>{{ projects[0].code }}</strong>
          </div>
          <div class="project-titleline">
            <p>{{ projects[0].name }}</p>
            <h3>{{ projects[0].title }}</h3>
          </div>
          <p class="project-summary">{{ projects[0].summary }}</p>
          <p class="project-detail">{{ projects[0].detail }}</p>
          <div class="role-row">
            <span>ROLE</span>
            <p>{{ projects[0].role }}</p>
          </div>
          <ul class="evidence-list">
            <li v-for="point in projects[0].points" :key="point">{{ point }}</li>
          </ul>
          <div class="tech-row">
            <span v-for="tech in projects[0].tech" :key="tech">{{ tech }}</span>
          </div>
        </article>

        <div class="support-projects">
          <article v-for="project in projects.slice(1)" :key="project.name" class="project-panel support-project reveal">
            <div class="record-head">
              <span>{{ project.level }}</span>
              <strong>{{ project.code }}</strong>
            </div>
            <h3>{{ project.name }}</h3>
            <p class="project-summary">{{ project.summary }}</p>
            <div class="role-row compact">
              <span>ROLE</span>
              <p>{{ project.role }}</p>
            </div>
            <ul class="evidence-list compact">
              <li v-for="point in project.points" :key="point">{{ point }}</li>
            </ul>
            <div class="tech-row">
              <span v-for="tech in project.tech" :key="tech">{{ tech }}</span>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section id="skills" class="content-section skills-section">
      <div class="section-heading reveal">
        <p class="section-index">03 / CAPABILITY MAP</p>
        <h2>技术栈与能力</h2>
        <p>围绕分布式系统、链上开发与工程实现形成的能力结构。</p>
      </div>

      <div class="pillar-grid">
        <article v-for="pillar in capabilityPillars" :key="pillar.title" class="capability-panel reveal">
          <h3>{{ pillar.title }}</h3>
          <p>{{ pillar.text }}</p>
          <div class="keyword-row">
            <span v-for="keyword in pillar.keywords" :key="keyword">{{ keyword }}</span>
          </div>
        </article>
      </div>

      <div class="skill-matrix reveal">
        <div v-for="group in skillGroups" :key="group.label" class="skill-group">
          <span>{{ group.label }}</span>
          <p>{{ group.items.join(' / ') }}</p>
        </div>
      </div>
      <p class="working-note reveal">具备需求分析、技术方案整理与文档输出能力，能够借助 AI 工具辅助信息处理与开发流程。</p>
    </section>

    <section id="timeline" class="content-section timeline-section">
      <div class="section-heading reveal">
        <p class="section-index">04 / GROWTH TRACE</p>
        <h2>成长时间线</h2>
        <p>从专业学习到项目实践，逐步形成面向系统与工程落地的能力方向。</p>
      </div>

      <div class="timeline-list">
        <article v-for="item in timeline" :key="item.title" class="timeline-record reveal">
          <span>{{ item.tag }}</span>
          <h3>{{ item.title }}</h3>
          <p>{{ item.text }}</p>
        </article>
      </div>
    </section>

    <section id="outside" class="content-section outside-section">
      <div class="section-heading reveal">
        <p class="section-index">05 / PERSONAL SIDE</p>
        <h2>工作之外</h2>
        <p>保持创造力、表达能力与长期学习节奏的个人侧面。</p>
      </div>

      <div class="outside-grid">
        <article v-for="item in outsideItems" :key="item.title" class="outside-panel reveal">
          <span>{{ item.label }}</span>
          <h3>{{ item.title }}</h3>
          <p>{{ item.text }}</p>
        </article>
      </div>
    </section>

    <section id="contact" class="contact-section">
      <div class="section-heading reveal">
        <p class="section-index">06 / CONTACT CHANNEL</p>
        <h2>联系我</h2>
        <p>如果你希望进一步了解项目、经历或合作可能，欢迎直接联系。</p>
      </div>

      <div class="contact-records reveal">
        <a v-for="item in contacts" :key="item.label" :href="item.href" class="contact-row" :target="item.label === 'GITHUB' ? '_blank' : undefined" rel="noreferrer">
          <span>{{ item.label }}</span>
          <strong>{{ item.value }}</strong>
        </a>
      </div>
    </section>
  </main>
</template>

<style>
:root {
  color-scheme: dark;
  --bg: #151817;
  --bg-soft: #1c211f;
  --bg-panel: rgba(37, 43, 40, 0.72);
  --bg-panel-strong: rgba(48, 55, 52, 0.9);
  --text: #eef3ef;
  --muted: #9aa59f;
  --quiet: #68746f;
  --line: rgba(219, 232, 222, 0.14);
  --line-strong: rgba(179, 201, 187, 0.28);
  --green: #42f06d;
  --green-soft: rgba(66, 240, 109, 0.18);
  --green-faint: rgba(66, 240, 109, 0.08);
  --metal: #9ca6a0;
  --metal-dark: #353d3a;
  --shadow: rgba(0, 0, 0, 0.36);
  --font-sans: "Noto Sans SC", "Microsoft YaHei", sans-serif;
  --font-mono: "JetBrains Mono", "SFMono-Regular", Consolas, monospace;
}

* {
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  min-width: 320px;
  background: var(--bg);
  color: var(--text);
  font-family: var(--font-sans);
}

a {
  color: inherit;
}

.portfolio-shell {
  min-height: 100vh;
  overflow: hidden;
  position: relative;
  background:
    linear-gradient(rgba(255, 255, 255, 0.025) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.018) 1px, transparent 1px),
    radial-gradient(circle at 70% 8%, rgba(66, 240, 109, 0.12), transparent 26%),
    linear-gradient(135deg, #1b201e 0%, #111412 52%, #202522 100%);
  background-size: 42px 42px, 42px 42px, auto, auto;
}

.page-noise {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
  opacity: 0.22;
  background-image:
    repeating-linear-gradient(0deg, rgba(255,255,255,0.025) 0 1px, transparent 1px 3px);
  mix-blend-mode: soft-light;
}

.site-nav {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 22px clamp(20px, 4vw, 64px);
  background: linear-gradient(180deg, rgba(21, 24, 23, 0.92), rgba(21, 24, 23, 0));
  backdrop-filter: blur(12px);
}

.brand-mark {
  font-family: var(--font-mono);
  font-size: 0.82rem;
  text-decoration: none;
  color: var(--green);
}

.nav-links {
  display: flex;
  align-items: center;
  gap: 24px;
}

.nav-links a {
  font-size: 0.86rem;
  color: var(--muted);
  text-decoration: none;
  transition: color 0.25s ease;
}

.nav-links a:hover {
  color: var(--text);
}

.hero-section,
.content-section,
.contact-section {
  position: relative;
  z-index: 2;
  width: min(1180px, calc(100% - 40px));
  margin: 0 auto;
}

.hero-section {
  min-height: 100vh;
  padding: 130px 0 44px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.hero-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.08fr) minmax(340px, 0.92fr);
  gap: clamp(32px, 6vw, 76px);
  align-items: center;
  flex: 1;
}

.hero-kicker,
.section-index,
.metric-cell span,
.record-head,
.role-row span,
.skill-group span,
.outside-panel span,
.timeline-record span,
.contact-row span,
.system-label,
.hero-name {
  font-family: var(--font-mono);
}

.hero-kicker {
  margin: 0 0 22px;
  color: var(--green);
  font-size: 0.82rem;
}

.hero-title {
  margin: 0;
  font-size: clamp(4rem, 12vw, 8.6rem);
  line-height: 0.9;
  font-weight: 700;
  color: var(--text);
}

.hero-name {
  margin: 22px 0 0;
  font-size: clamp(1.1rem, 2vw, 1.45rem);
  color: var(--metal);
}

.hero-role {
  margin: 42px 0 0;
  max-width: 660px;
  font-size: clamp(1.12rem, 2vw, 1.45rem);
  color: var(--text);
}

.hero-copy {
  max-width: 620px;
  margin: 18px 0 0;
  color: var(--muted);
  font-size: 1rem;
  line-height: 1.9;
}

.hero-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  margin-top: 34px;
}

.primary-action,
.secondary-action {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 44px;
  padding: 0 20px;
  border: 1px solid var(--line-strong);
  border-radius: 6px;
  text-decoration: none;
  font-size: 0.92rem;
  transition: border-color 0.25s ease, color 0.25s ease, background 0.25s ease;
}

.primary-action {
  color: #07110b;
  border-color: var(--green);
  background: var(--green);
}

.secondary-action {
  color: var(--text);
  background: rgba(255, 255, 255, 0.03);
}

.secondary-action:hover {
  border-color: var(--green);
  color: var(--green);
}

.core-visual {
  min-height: 560px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.core-frame {
  position: relative;
  width: min(420px, 100%);
  height: 560px;
  border: 1px solid var(--line);
  border-radius: 8px;
  background:
    linear-gradient(90deg, rgba(255,255,255,0.04), transparent 18%, transparent 82%, rgba(255,255,255,0.035)),
    linear-gradient(180deg, rgba(66, 240, 109, 0.08), rgba(255,255,255,0.025) 36%, rgba(66, 240, 109, 0.05));
  box-shadow: 0 28px 80px var(--shadow), inset 0 0 70px rgba(255,255,255,0.035);
}

.core-frame::before,
.core-frame::after {
  content: '';
  position: absolute;
  left: 32px;
  right: 32px;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--line-strong), transparent);
}

.core-frame::before {
  top: 76px;
}

.core-frame::after {
  bottom: 76px;
}

.core-shell {
  position: absolute;
  inset: 70px 126px;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  filter: drop-shadow(0 28px 42px rgba(0,0,0,0.45));
}

.core-cap {
  height: 62px;
  border: 1px solid var(--line-strong);
  background:
    linear-gradient(115deg, #65716b, #2c3430 42%, #a8b2ab 58%, #3a433f);
  box-shadow: inset 0 0 24px rgba(255,255,255,0.08);
}

.core-cap-top {
  border-radius: 8px 8px 3px 3px;
}

.core-cap-bottom {
  border-radius: 3px 3px 8px 8px;
}

.core-body {
  position: relative;
  flex: 1;
  border-left: 1px solid var(--line-strong);
  border-right: 1px solid var(--line-strong);
  background:
    linear-gradient(90deg, #2b3330, #4a5550 18%, #1c2420 48%, #68736d 52%, #2a3330 76%, #111714);
  overflow: hidden;
}

.core-glass {
  --core-glow: 0.42;
  position: absolute;
  inset: 34px 28px;
  border: 1px solid rgba(66, 240, 109, 0.55);
  border-radius: 8px;
  background:
    linear-gradient(180deg, rgba(66, 240, 109, calc(var(--core-glow) * 0.22)), rgba(66, 240, 109, 0.035)),
    rgba(8, 20, 12, 0.48);
  overflow: hidden;
  box-shadow:
    0 0 calc(var(--core-glow) * 28px) rgba(66, 240, 109, 0.38),
    inset 0 0 34px rgba(66, 240, 109, 0.14);
}

.energy-column {
  position: absolute;
  top: 18px;
  bottom: 18px;
  left: 50%;
  width: 12px;
  transform: translateX(-50%);
  background: linear-gradient(180deg, transparent, rgba(66,240,109,0.82), transparent);
  box-shadow: 0 0 28px rgba(66,240,109,0.45);
}

.scan-blade {
  position: absolute;
  top: -30%;
  left: 0;
  right: 0;
  height: 34%;
  opacity: 0;
  background: linear-gradient(180deg, transparent, rgba(66,240,109,0.35), transparent);
}

.panel-line {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
  background: rgba(255,255,255,0.14);
}

.panel-line-a {
  left: 22%;
}

.panel-line-b {
  right: 22%;
}

.lock-node {
  position: absolute;
  left: 50%;
  width: 40px;
  height: 10px;
  border: 1px solid var(--line-strong);
  background: var(--metal-dark);
  transform: translateX(-50%);
}

.node-top {
  top: 14px;
}

.node-bottom {
  bottom: 14px;
}

.stabilizer {
  position: absolute;
  top: 180px;
  width: 78px;
  height: 190px;
  border: 1px solid var(--line-strong);
  background: linear-gradient(180deg, rgba(255,255,255,0.08), rgba(255,255,255,0.01));
}

.stabilizer-left {
  left: 64px;
  border-right: 0;
}

.stabilizer-right {
  right: 64px;
  border-left: 0;
}

.signal-rail {
  position: absolute;
  left: 72px;
  right: 72px;
  bottom: 48px;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--green-soft), transparent);
}

.signal-dot {
  position: absolute;
  left: 0;
  top: -3px;
  width: 7px;
  height: 7px;
  opacity: 0;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 16px rgba(66,240,109,0.7);
}

.system-label {
  position: absolute;
  left: 28px;
  bottom: 18px;
  font-size: 0.68rem;
  color: var(--quiet);
}

.metric-strip {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
  background: rgba(255,255,255,0.025);
}

.metric-cell {
  min-height: 82px;
  padding: 18px 20px;
  border-right: 1px solid var(--line);
}

.metric-cell:last-child {
  border-right: 0;
}

.metric-cell span {
  display: block;
  color: var(--quiet);
  font-size: 0.72rem;
  margin-bottom: 9px;
}

.metric-cell strong {
  display: block;
  color: var(--text);
  font-size: clamp(1rem, 1.8vw, 1.35rem);
  font-weight: 500;
}

.content-section,
.contact-section {
  padding: 112px 0;
}

.section-heading {
  max-width: 760px;
  margin-bottom: 44px;
}

.section-index {
  margin: 0 0 14px;
  color: var(--green);
  font-size: 0.78rem;
}

.section-heading h2 {
  margin: 0;
  font-size: clamp(2rem, 5vw, 4rem);
  line-height: 1.08;
  font-weight: 700;
}

.section-heading p:last-child {
  margin: 18px 0 0;
  color: var(--muted);
  line-height: 1.9;
}

.project-grid {
  display: grid;
  grid-template-columns: minmax(0, 1.15fr) minmax(320px, 0.85fr);
  gap: 22px;
}

.support-projects {
  display: grid;
  gap: 22px;
}

.project-panel,
.capability-panel,
.timeline-record,
.outside-panel {
  border: 1px solid var(--line);
  border-radius: 8px;
  background: var(--bg-panel);
  box-shadow: 0 18px 60px rgba(0,0,0,0.22);
  transition: border-color 0.25s ease, transform 0.25s ease, background 0.25s ease;
}

.project-panel:hover,
.capability-panel:hover,
.timeline-record:hover,
.outside-panel:hover {
  border-color: rgba(66,240,109,0.42);
  transform: translateY(-3px);
  background: var(--bg-panel-strong);
}

.project-panel {
  padding: clamp(24px, 4vw, 38px);
}

.primary-project {
  min-height: 100%;
  background:
    linear-gradient(135deg, rgba(66,240,109,0.1), transparent 36%),
    var(--bg-panel);
}

.record-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 28px;
  color: var(--quiet);
  font-size: 0.74rem;
}

.record-head strong {
  color: var(--green);
  font-size: 1.1rem;
  font-weight: 500;
}

.project-titleline p {
  margin: 0 0 8px;
  color: var(--green);
  font-family: var(--font-mono);
  font-size: 0.92rem;
}

.project-panel h3,
.capability-panel h3,
.timeline-record h3,
.outside-panel h3 {
  margin: 0;
  font-size: 1.35rem;
  font-weight: 600;
}

.project-summary,
.project-detail {
  margin: 18px 0 0;
  color: var(--muted);
  line-height: 1.85;
}

.project-detail {
  color: var(--text);
}

.role-row {
  display: grid;
  grid-template-columns: 72px 1fr;
  gap: 18px;
  margin-top: 26px;
  padding: 16px 0;
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
}

.role-row.compact {
  grid-template-columns: 58px 1fr;
  margin-top: 18px;
  padding: 12px 0;
}

.role-row span {
  color: var(--green);
  font-size: 0.72rem;
}

.role-row p {
  margin: 0;
  color: var(--text);
  line-height: 1.7;
}

.evidence-list {
  list-style: none;
  margin: 24px 0 0;
  padding: 0;
  display: grid;
  gap: 14px;
}

.evidence-list.compact {
  gap: 10px;
}

.evidence-list li {
  position: relative;
  padding-left: 18px;
  color: var(--muted);
  line-height: 1.75;
}

.evidence-list li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0.76em;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 12px rgba(66,240,109,0.55);
}

.tech-row,
.keyword-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 24px;
}

.tech-row span,
.keyword-row span {
  padding: 7px 10px;
  border: 1px solid rgba(66,240,109,0.22);
  border-radius: 6px;
  color: var(--green);
  background: var(--green-faint);
  font-family: var(--font-mono);
  font-size: 0.74rem;
}

.pillar-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 20px;
}

.capability-panel {
  padding: 28px;
}

.capability-panel p {
  margin: 16px 0 0;
  color: var(--muted);
  line-height: 1.8;
}

.skill-matrix {
  margin-top: 26px;
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
  background: rgba(255,255,255,0.025);
}

.skill-group {
  padding: 20px;
  border-right: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
}

.skill-group:nth-child(2n) {
  border-right: 0;
}

.skill-group:nth-last-child(-n + 2) {
  border-bottom: 0;
}

.skill-group span {
  color: var(--green);
  font-size: 0.78rem;
}

.skill-group p {
  margin: 8px 0 0;
  color: var(--muted);
  line-height: 1.7;
}

.working-note {
  margin: 24px 0 0;
  padding: 22px 24px;
  border-left: 3px solid var(--green);
  color: var(--text);
  background: rgba(66,240,109,0.06);
  line-height: 1.85;
}

.timeline-list {
  position: relative;
  display: grid;
  gap: 18px;
  max-width: 900px;
}

.timeline-list::before {
  content: '';
  position: absolute;
  top: 8px;
  bottom: 8px;
  left: 11px;
  width: 1px;
  background: linear-gradient(180deg, var(--green), var(--line), transparent);
}

.timeline-record {
  position: relative;
  margin-left: 42px;
  padding: 24px 28px;
}

.timeline-record::before {
  content: '';
  position: absolute;
  left: -37px;
  top: 30px;
  width: 13px;
  height: 13px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 16px rgba(66,240,109,0.55);
}

.timeline-record span,
.outside-panel span {
  display: block;
  margin-bottom: 10px;
  color: var(--green);
  font-size: 0.74rem;
}

.timeline-record p,
.outside-panel p {
  margin: 12px 0 0;
  color: var(--muted);
  line-height: 1.85;
}

.outside-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 20px;
}

.outside-panel {
  padding: 28px;
  min-height: 240px;
  background:
    linear-gradient(180deg, rgba(255,255,255,0.04), transparent),
    var(--bg-panel);
}

.contact-section {
  padding-bottom: 90px;
}

.contact-records {
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
  background: rgba(255,255,255,0.025);
}

.contact-row {
  display: grid;
  grid-template-columns: 160px 1fr;
  gap: 20px;
  padding: 22px 24px;
  text-decoration: none;
  border-bottom: 1px solid var(--line);
  transition: background 0.25s ease, color 0.25s ease;
}

.contact-row:last-child {
  border-bottom: 0;
}

.contact-row span {
  color: var(--green);
  font-size: 0.78rem;
}

.contact-row strong {
  color: var(--text);
  font-weight: 500;
  overflow-wrap: anywhere;
}

.contact-row:hover {
  background: var(--green-faint);
}

@media (max-width: 960px) {
  .nav-links {
    display: none;
  }

  .hero-layout,
  .project-grid,
  .pillar-grid,
  .outside-grid {
    grid-template-columns: 1fr;
  }

  .hero-section {
    padding-top: 104px;
  }

  .core-visual {
    min-height: 460px;
  }

  .core-frame {
    height: 460px;
  }

  .core-shell {
    inset: 54px 34%;
  }

  .stabilizer-left {
    left: 18%;
  }

  .stabilizer-right {
    right: 18%;
  }

  .metric-strip {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    margin-top: 34px;
  }

  .metric-cell:nth-child(2n) {
    border-right: 0;
  }

  .metric-cell:nth-child(-n + 2) {
    border-bottom: 1px solid var(--line);
  }
}

@media (max-width: 640px) {
  .hero-section,
  .content-section,
  .contact-section {
    width: min(100% - 28px, 1180px);
  }

  .site-nav {
    padding: 18px 16px;
  }

  .hero-title {
    font-size: clamp(3.6rem, 23vw, 5.6rem);
  }

  .hero-role {
    margin-top: 30px;
  }

  .core-visual {
    min-height: 380px;
  }

  .core-frame {
    height: 380px;
  }

  .core-shell {
    inset: 46px 31%;
  }

  .core-cap {
    height: 48px;
  }

  .stabilizer {
    top: 132px;
    height: 132px;
    width: 54px;
  }

  .signal-rail {
    left: 44px;
    right: 44px;
  }

  .metric-strip,
  .skill-matrix {
    grid-template-columns: 1fr;
  }

  .metric-cell,
  .skill-group {
    border-right: 0;
    border-bottom: 1px solid var(--line);
  }

  .metric-cell:last-child,
  .skill-group:last-child {
    border-bottom: 0;
  }

  .project-panel,
  .capability-panel,
  .timeline-record,
  .outside-panel {
    padding: 22px;
  }

  .role-row,
  .role-row.compact,
  .contact-row {
    grid-template-columns: 1fr;
    gap: 8px;
  }
}
</style>
