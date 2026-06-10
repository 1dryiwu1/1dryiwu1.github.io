<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue';
import { gsap } from 'gsap';

const root = ref(null);
const networkCanvas = ref(null);
const introActive = ref(true);
const navFloating = ref(false);
const activeProjectIndex = ref(0);
const activeCapabilityIndex = ref(0);
let ctx;
let revealObserver;
let animationFrame;
let resizeHandler;
let navScrollHandler;
let networkVisibilityObserver;
let documentVisibilityHandler;
let introTimeline;
let introFallbackTimer;
let glowCleanup = [];
let introAmbientTweens = [];
let networkCanvasReady = false;

const navItems = [
  { label: '入口', href: '#hero' },
  { label: '档案', href: '#profile' },
  { label: '项目', href: '#projects' },
  { label: '能力', href: '#skills' },
  { label: '路径', href: '#timeline' },
  { label: '联系', href: '#contact' },
];

const heroStats = [
  { label: 'PROJECTS', value: '03' },
  { label: 'CONSENSUS', value: 'RAFT' },
  { label: 'SMART CONTRACT', value: 'SOLIDITY' },
  { label: 'FRONTEND', value: 'VUE 3' },
];

const chainBlocks = [
  { title: 'GENESIS', hash: '0x9F2A...B71C', meta: '可信数据协作' },
  { title: 'CONSENSUS', hash: '0x42E8...19AD', meta: 'Raft 日志复制' },
  { title: 'CONTRACT', hash: '0xB6C1...70F3', meta: '链上规则执行' },
];

const projects = [
  {
    code: '01',
    label: '主项目',
    title: 'MedTrust-Raft 医疗数据联盟治理平台',
    summary: '面向多医疗机构可信数据共享场景，构建以 Raft 共识为核心的联盟治理平台。',
    role: '主导系统设计，并独立完成 Raft 核心机制实现。',
    points: [
      '从零实现 Leader 选举、日志复制、心跳维护等关键共识流程。',
      '围绕多机构协作场景设计联盟治理架构，提升数据共享过程中的可信度与一致性。',
      '完成从共识层到服务层的工程串联，让系统具备可运行、可演示的完整形态。',
    ],
    tech: ['Go', 'Raft', 'Gin', 'BoltDB', 'Vue 3'],
  },
  {
    code: '02',
    label: '链上实践',
    title: '公益众筹智能合约',
    summary: '基于以太坊的去中心化众筹合约实践，聚焦资金释放与流程透明。',
    role: '负责合约逻辑设计与关键安全机制实现。',
    points: [
      '实现目标验证、自动拨款、失败退款等核心众筹逻辑。',
      '按里程碑拆分资金释放流程，增强资金使用的可控性。',
      '在合约设计中引入常见安全模式，降低链上执行风险。',
    ],
    tech: ['Solidity', 'Ethereum', 'Truffle', 'Ganache'],
  },
  {
    code: '03',
    label: '工程表达',
    title: 'AI 工具与技术文档实践',
    summary: '将 AI 工具、需求分析与技术文档能力结合到开发流程中。',
    role: '负责方案整理、技术表达与辅助研发流程优化。',
    points: [
      '把复杂需求拆解为结构化方案，提高沟通与执行效率。',
      '持续输出需求分析、技术方案与架构说明文档。',
      '借助 AI 工具辅助信息整理、内容生成与开发支持。',
    ],
    tech: ['AI Tools', 'Technical Writing', 'Solution Design'],
  },
];

const capabilityPillars = [
  {
    title: '分布式系统',
    text: '围绕一致性机制、服务协作与系统设计建立技术主轴。',
    keywords: ['Raft', '一致性机制', '日志复制', '服务协作'],
  },
  {
    title: '区块链开发',
    text: '理解链上执行逻辑，能够完成智能合约设计、测试与风险控制。',
    keywords: ['Solidity', 'Ethereum', '智能合约', '链上逻辑'],
  },
  {
    title: '工程落地',
    text: '关注从方案、接口、数据存储到前端表达的完整交付过程。',
    keywords: ['Go', 'Vue 3', 'Gin', 'BoltDB'],
  },
];

const skillGroups = [
  { label: 'Languages', items: 'Go / Solidity / Python / JavaScript' },
  { label: 'Backend & Infra', items: 'Gin / BoltDB / 共识机制实现 / 服务架构' },
  { label: 'Frontend', items: 'Vue 3 / 交互实现 / 页面工程化' },
  { label: 'Blockchain', items: 'Ethereum / Truffle / Ganache / 合约开发' },
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
    text: '通过比赛和系统训练补强问题拆解、逻辑推演与实现能力。',
  },
  {
    tag: 'STAGE 03',
    title: '项目实践阶段',
    text: '把学习内容落到真实项目中，重点进入联盟治理、共识机制、智能合约等更成体系的实践。',
  },
  {
    tag: 'STAGE 04',
    title: '工程表达与能力收束',
    text: '开始关注架构、方案、文档、协作和交付质量，形成更完整的工程思维。',
  },
];

const certificates = [
  { title: '区块链技术应用创新大赛 · 一等奖', meta: '本科组 / 第四届', date: '2025.12' },
  { title: '全国大学生数学竞赛 · 三等奖', meta: '非数学 A 类 / 第十七届', date: '2025.12' },
  { title: '挑战杯校赛 · 三等奖', meta: '第十五届 / 红蓝维安', date: '2026.03' },
];

const contacts = [
  { label: 'EMAIL', value: '2489555099@qq.com', href: 'mailto:2489555099@qq.com' },
  { label: 'PHONE', value: '17883563946', href: 'tel:17883563946' },
  { label: 'GITHUB', value: 'github.com/1dryiwu1', href: 'https://github.com/1dryiwu1' },
];

const activeProject = computed(() => projects[activeProjectIndex.value]);
const activeCapability = computed(() => capabilityPillars[activeCapabilityIndex.value]);
const projectTabStyle = computed(() => ({ '--active-tab-index': activeProjectIndex.value }));
const capabilityTabStyle = computed(() => ({ '--active-tab-index': activeCapabilityIndex.value }));

function closeIntro() {
  introTimeline?.progress(1);
  finishIntro();
}

function killIntroAmbientTweens() {
  introAmbientTweens.forEach((tween) => tween.kill());
  introAmbientTweens = [];
}

function finishIntro() {
  if (introFallbackTimer) {
    window.clearTimeout(introFallbackTimer);
    introFallbackTimer = undefined;
  }
  killIntroAmbientTweens();
  introActive.value = false;
  setupNetworkCanvas();
}

function animatePanel(selector) {
  nextTick(() => {
    const target = root.value?.querySelector(selector);
    if (!target) return;
    const children = target.querySelectorAll('.record-head, h3, .project-summary, .role-row, .evidence-list li, .tech-row span, .capability-label, p, .keyword-row span, .signal-meter span');
    const timeline = gsap.timeline();

    timeline.fromTo(
      target,
      { autoAlpha: 0, y: 14 },
      { autoAlpha: 1, y: 0, duration: 0.34, ease: 'power3.out' },
    );

    if (children.length) {
      timeline.fromTo(
        children,
        { autoAlpha: 0, y: 10 },
        { autoAlpha: 1, y: 0, duration: 0.34, ease: 'power3.out', stagger: 0.035 },
        '-=0.18',
      );
    }
  });
}

function selectProject(index) {
  if (activeProjectIndex.value === index) return;
  activeProjectIndex.value = index;
  animatePanel('.project-focus');
}

function selectCapability(index) {
  if (activeCapabilityIndex.value === index) return;
  activeCapabilityIndex.value = index;
  animatePanel('.capability-focus');
}

function setupNetworkCanvas() {
  if (networkCanvasReady) return;
  const canvas = networkCanvas.value;
  if (!canvas) return;

  const context = canvas.getContext('2d');
  if (!context) return;
  networkCanvasReady = true;

  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  let width = 0;
  let height = 0;
  let dpr = 1;
  let tick = 0;
  let nodes = [];
  let isVisible = true;
  const frameInterval = 1000 / 30;
  let lastFrame = -frameInterval;

  const resize = () => {
    const rect = canvas.getBoundingClientRect();
    dpr = Math.min(window.devicePixelRatio || 1, 1.5);
    width = rect.width;
    height = rect.height;
    if (!width || !height) return;
    canvas.width = Math.floor(width * dpr);
    canvas.height = Math.floor(height * dpr);
    context.setTransform(dpr, 0, 0, dpr, 0, 0);

    const count = width < 520 ? 14 : 22;
    nodes = Array.from({ length: count }, (_, index) => ({
      x: ((index * 89) % Math.max(width, 1)) + Math.random() * 42,
      y: ((index * 137) % Math.max(height, 1)) + Math.random() * 36,
      vx: (Math.random() - 0.5) * 0.22,
      vy: (Math.random() - 0.5) * 0.22,
      r: index % 5 === 0 ? 4.2 : 2.8,
      phase: Math.random() * Math.PI * 2,
    }));
  };

  const draw = (time = 0) => {
    if (!reduceMotion) animationFrame = requestAnimationFrame(draw);
    if (!isVisible || document.hidden || time - lastFrame < frameInterval || !width || !height) return;
    lastFrame = time;

    tick += reduceMotion ? 0 : 0.012;
    context.clearRect(0, 0, width, height);

    const gradient = context.createLinearGradient(0, 0, width, height);
    gradient.addColorStop(0, 'rgba(183, 229, 186, 0.12)');
    gradient.addColorStop(0.55, 'rgba(92, 168, 124, 0.10)');
    gradient.addColorStop(1, 'rgba(6, 133, 98, 0.07)');
    context.fillStyle = gradient;
    context.fillRect(0, 0, width, height);

    nodes.forEach((node) => {
      if (!reduceMotion) {
        node.x += node.vx;
        node.y += node.vy;
        if (node.x < 10 || node.x > width - 10) node.vx *= -1;
        if (node.y < 10 || node.y > height - 10) node.vy *= -1;
      }
    });

    for (let i = 0; i < nodes.length; i += 1) {
      for (let j = i + 1; j < nodes.length; j += 1) {
        const a = nodes[i];
        const b = nodes[j];
        const distance = Math.hypot(a.x - b.x, a.y - b.y);
        if (distance > 142) continue;
        const alpha = (1 - distance / 142) * 0.26;
        context.strokeStyle = `rgba(168, 255, 207, ${alpha})`;
        context.lineWidth = 1;
        context.beginPath();
        context.moveTo(a.x, a.y);
        context.lineTo(b.x, b.y);
        context.stroke();
      }
    }

    nodes.forEach((node) => {
      const pulse = reduceMotion ? 0.35 : 0.35 + Math.sin(tick * 3 + node.phase) * 0.18;
      context.fillStyle = `rgba(183, 229, 186, ${0.58 + pulse * 0.32})`;
      context.shadowColor = 'rgba(183, 229, 186, 0.46)';
      context.shadowBlur = 12;
      context.beginPath();
      context.arc(node.x, node.y, node.r + pulse, 0, Math.PI * 2);
      context.fill();
      context.shadowBlur = 0;
    });

  };

  resize();
  draw();
  resizeHandler = resize;
  window.addEventListener('resize', resizeHandler);

  if ('IntersectionObserver' in window) {
    networkVisibilityObserver = new IntersectionObserver(([entry]) => {
      isVisible = entry.isIntersecting;
    }, { threshold: 0.05 });
    networkVisibilityObserver.observe(canvas);
  }

  documentVisibilityHandler = () => {
    isVisible = !document.hidden;
  };
  document.addEventListener('visibilitychange', documentVisibilityHandler);
}

function setupRevealAnimations() {
  const items = root.value.querySelectorAll('.reveal');
  revealObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (!entry.isIntersecting) return;
        gsap.to(entry.target, {
          autoAlpha: 1,
          y: 0,
          duration: 0.72,
          ease: 'power3.out',
        });
        revealObserver.unobserve(entry.target);
      });
    },
    { threshold: 0.16, rootMargin: '0px 0px -10% 0px' },
  );
  items.forEach((item) => revealObserver.observe(item));
}

function setupModuleGlow() {
  const glowTargets = root.value?.querySelectorAll([
    '.network-visual',
    '.stat-card',
    '.profile-tags span',
    '.project-workbench',
    '.capability-console',
    '.project-focus',
    '.capability-focus',
    '.timeline-record',
    '.certificate-card',
    '.skill-matrix',
    '.contact-records',
    '.chain-card',
    '.network-console',
  ].join(','));

  if (!glowTargets?.length) return;

  glowTargets.forEach((target) => {
    target.classList.add('module-glow');
    if (!target.querySelector(':scope > .module-edge-light')) {
      const light = document.createElement('span');
      light.className = 'module-edge-light';
      light.setAttribute('aria-hidden', 'true');
      target.appendChild(light);
    }

    let rect = target.getBoundingClientRect();
    let pendingFrame = 0;
    let pendingEvent = null;

    const refreshRect = () => {
      rect = target.getBoundingClientRect();
    };

    const updateGlow = () => {
      pendingFrame = 0;
      if (!pendingEvent || !rect.width || !rect.height) return;
      const x = pendingEvent.clientX - rect.left;
      const y = pendingEvent.clientY - rect.top;
      const cx = rect.width / 2;
      const cy = rect.height / 2;
      const dx = x - cx;
      const dy = y - cy;
      const kx = dx === 0 ? Infinity : cx / Math.abs(dx);
      const ky = dy === 0 ? Infinity : cy / Math.abs(dy);
      const edge = Math.min(Math.max(1 / Math.min(kx, ky), 0), 1);
      let angle = Math.atan2(dy, dx) * (180 / Math.PI) + 90;
      if (angle < 0) angle += 360;

      target.style.setProperty('--edge-proximity', (edge * 100).toFixed(3));
      target.style.setProperty('--cursor-angle', `${angle.toFixed(3)}deg`);
      target.style.setProperty('--pointer-x', `${x.toFixed(1)}px`);
      target.style.setProperty('--pointer-y', `${y.toFixed(1)}px`);
    };

    const handlePointerMove = (event) => {
      pendingEvent = event;
      if (!pendingFrame) pendingFrame = requestAnimationFrame(updateGlow);
    };

    const handlePointerLeave = () => {
      pendingEvent = null;
      if (pendingFrame) {
        cancelAnimationFrame(pendingFrame);
        pendingFrame = 0;
      }
      target.style.setProperty('--edge-proximity', '0');
    };

    target.addEventListener('pointerenter', refreshRect);
    target.addEventListener('pointermove', handlePointerMove);
    target.addEventListener('pointerleave', handlePointerLeave);
    window.addEventListener('resize', refreshRect);
    glowCleanup.push(() => {
      if (pendingFrame) cancelAnimationFrame(pendingFrame);
      target.removeEventListener('pointerenter', refreshRect);
      target.removeEventListener('pointermove', handlePointerMove);
      target.removeEventListener('pointerleave', handlePointerLeave);
      window.removeEventListener('resize', refreshRect);
    });
  });
}

function setupFloatingNav() {
  let ticking = false;

  const updateNavState = () => {
    ticking = false;
    const shouldFloat = window.scrollY > window.innerHeight * 0.86;
    if (navFloating.value !== shouldFloat) navFloating.value = shouldFloat;
  };

  const requestNavUpdate = () => {
    if (ticking) return;
    ticking = true;
    requestAnimationFrame(updateNavState);
  };

  navScrollHandler = requestNavUpdate;
  updateNavState();
  window.addEventListener('scroll', navScrollHandler, { passive: true });
  window.addEventListener('resize', navScrollHandler);
}

onMounted(() => {
  if (!root.value) return;

  ctx = gsap.context(() => {
    const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    gsap.set('.reveal', { autoAlpha: 0, y: 26 });

    if (reduceMotion) {
      gsap.set('.intro-overlay', { autoAlpha: 0, pointerEvents: 'none' });
      finishIntro();
    } else {
      introFallbackTimer = window.setTimeout(finishIntro, 5500);
      introTimeline = gsap.timeline({
        defaults: { ease: 'power3.out' },
        onComplete: finishIntro,
      });

      introTimeline
        .set('.site-nav, .hero-kicker, .hero-title, .hero-copy, .hero-actions, .stat-card, .chain-card, .network-console', { autoAlpha: 0 })
        .from('.intro-gridline', { scaleX: 0, transformOrigin: 'left center', duration: 0.7, stagger: 0.08 })
        .from('.intro-access-frame', { clipPath: 'inset(48% 0 48% 0)', autoAlpha: 0, duration: 0.72, ease: 'power3.inOut' }, '-=0.35')
        .from('.intro-corner', { scale: 0, autoAlpha: 0, duration: 0.28, stagger: 0.04 }, '-=0.45')
        .from('.intro-orbit span', { scale: 0.78, autoAlpha: 0, duration: 0.5, stagger: 0.08 }, '-=0.35')
        .from('.intro-readout li', { x: -18, autoAlpha: 0, duration: 0.38, stagger: 0.08 }, '-=0.28')
        .from('.intro-biometric span, .intro-dna span, .intro-number-cloud span', {
          autoAlpha: 0,
          y: 10,
          duration: 0.34,
          stagger: { each: 0.025, from: 'random' },
        }, '-=0.32')
        .from('.intro-node', { scale: 0, autoAlpha: 0, duration: 0.36, stagger: { each: 0.06, from: 'center' } }, '-=0.2')
        .from('.intro-title-line', { yPercent: 105, autoAlpha: 0, duration: 0.72, stagger: 0.08 }, '-=0.08')
        .from('.intro-copy, .intro-status', { autoAlpha: 0, y: 14, duration: 0.5, stagger: 0.08 }, '-=0.3')
        .to('.intro-split.top', { yPercent: -102, duration: 0.78, ease: 'power4.inOut' }, 'reveal+=0.35')
        .to('.intro-split.bottom', { yPercent: 102, duration: 0.78, ease: 'power4.inOut' }, 'reveal+=0.35')
        .to('.intro-stage', { scale: 1.06, autoAlpha: 0, duration: 0.62, ease: 'power2.inOut' }, 'reveal+=0.44')
        .to('.intro-overlay', { autoAlpha: 0, pointerEvents: 'none', duration: 0.25 }, 'reveal+=0.9')
        .fromTo('.site-nav', { autoAlpha: 0, y: -14 }, { autoAlpha: 1, y: 0, duration: 0.55 }, 'reveal+=0.78')
        .fromTo('.hero-kicker, .hero-title, .hero-copy, .hero-actions', { autoAlpha: 0, y: 24 }, {
          autoAlpha: 1,
          y: 0,
          duration: 0.68,
          stagger: 0.08,
        }, 'reveal+=0.95')
        .fromTo('.stat-card, .chain-card, .network-console', { autoAlpha: 0, y: 18 }, {
          autoAlpha: 1,
          y: 0,
          duration: 0.55,
          stagger: 0.06,
        }, 'reveal+=1.1');
    }

    if (!reduceMotion) {
      gsap.to('.tx-line', {
        xPercent: 120,
        duration: 2.2,
        repeat: -1,
        repeatDelay: 1.2,
        ease: 'power2.inOut',
        stagger: 0.28,
      });

      gsap.to('.chain-card', {
        y: (index) => (index % 2 === 0 ? -7 : 7),
        duration: 3.4,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: 0.25,
      });

      introAmbientTweens.push(gsap.to('.intro-pulse', {
        scale: 1.08,
        autoAlpha: 0.72,
        duration: 1.6,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
      }));

      introAmbientTweens.push(gsap.to('.intro-orbit', {
        rotate: 360,
        duration: 18,
        repeat: -1,
        ease: 'none',
      }));

      introAmbientTweens.push(gsap.to('.intro-biometric span', {
        scaleX: () => gsap.utils.random(0.35, 1),
        duration: 0.72,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: 0.05,
      }));

      introAmbientTweens.push(gsap.to('.intro-number-cloud span', {
        y: (index) => (index % 2 === 0 ? -8 : 8),
        autoAlpha: 0.42,
        duration: 1.7,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: 0.12,
      }));

      gsap.fromTo('.stat-card strong', {
        y: 8,
        autoAlpha: 0.45,
      }, {
        y: 0,
        autoAlpha: 1,
        duration: 0.64,
        repeat: -1,
        repeatDelay: 4.2,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: { each: 0.08, from: 'center' },
      });

      gsap.to('.signal-meter span', {
        scaleY: () => gsap.utils.random(0.35, 1),
        duration: 0.74,
        repeat: -1,
        yoyo: true,
        ease: 'sine.inOut',
        stagger: 0.06,
      });
    }

    setupRevealAnimations();
    setupModuleGlow();
    setupFloatingNav();
  }, root.value);
});

onUnmounted(() => {
  revealObserver?.disconnect();
  networkVisibilityObserver?.disconnect();
  ctx?.revert();
  killIntroAmbientTweens();
  glowCleanup.forEach((cleanup) => cleanup());
  glowCleanup = [];
  if (animationFrame) cancelAnimationFrame(animationFrame);
  if (introFallbackTimer) window.clearTimeout(introFallbackTimer);
  if (resizeHandler) window.removeEventListener('resize', resizeHandler);
  if (documentVisibilityHandler) document.removeEventListener('visibilitychange', documentVisibilityHandler);
  if (navScrollHandler) {
    window.removeEventListener('scroll', navScrollHandler);
    window.removeEventListener('resize', navScrollHandler);
  }
});
</script>

<template>
  <main ref="root" class="portfolio-shell">
    <section v-if="introActive" class="intro-overlay" aria-label="站点开场动画">
      <div class="intro-split top"></div>
      <div class="intro-split bottom"></div>
      <div class="intro-stage">
        <div class="intro-grid" aria-hidden="true">
          <span class="intro-gridline"></span>
          <span class="intro-gridline"></span>
          <span class="intro-gridline"></span>
          <span class="intro-gridline"></span>
        </div>
        <div class="intro-access-frame" aria-hidden="true">
          <span class="intro-corner c1"></span>
          <span class="intro-corner c2"></span>
          <span class="intro-corner c3"></span>
          <span class="intro-corner c4"></span>
          <div class="intro-orbit">
            <span></span>
            <span></span>
            <span></span>
          </div>
          <div class="intro-biometric">
            <span v-for="index in 12" :key="index"></span>
          </div>
          <div class="intro-geo-trace">
            <span class="trace-node a"></span>
            <span class="trace-node b"></span>
            <span class="trace-node c"></span>
            <span class="trace-path p1"></span>
            <span class="trace-path p2"></span>
          </div>
        </div>
        <ul class="intro-readout" aria-hidden="true">
          <li><span>ACCESS</span><strong>REQUEST ACCEPTED</strong></li>
          <li><span>ID</span><strong>CONFIRMED</strong></li>
          <li><span>CHAIN</span><strong>GENESIS VERIFIED</strong></li>
        </ul>
        <div class="intro-number-cloud" aria-hidden="true">
          <span>0924</span>
          <span>17E8</span>
          <span>RAFT</span>
          <span>0x42</span>
          <span>SYNC</span>
          <span>2026</span>
        </div>
        <div class="intro-dna" aria-hidden="true">
          <span v-for="index in 8" :key="index"></span>
        </div>
        <div class="intro-constellation" aria-hidden="true">
          <span class="intro-node n1"></span>
          <span class="intro-node n2"></span>
          <span class="intro-node n3"></span>
          <span class="intro-node n4"></span>
          <span class="intro-node n5"></span>
          <span class="intro-node n6"></span>
          <span class="intro-link l1"></span>
          <span class="intro-link l2"></span>
          <span class="intro-link l3"></span>
          <span class="intro-pulse"></span>
        </div>
        <p class="intro-status">NODE SYNC / CONSENSUS READY / CONTRACT ROUTED</p>
        <h1 class="intro-title" aria-label="可信系统正在启动">
          <span class="intro-title-mask"><span class="intro-title-line">TRUSTED</span></span>
          <span class="intro-title-mask"><span class="intro-title-line">SYSTEMS</span></span>
          <span class="intro-title-mask accent"><span class="intro-title-line">ONLINE</span></span>
        </h1>
        <p class="intro-copy">A blockchain engineering portfolio is being initialized.</p>
        <button class="intro-skip" type="button" @click="closeIntro">进入</button>
      </div>
    </section>

    <nav class="site-nav" :class="{ 'is-floating': navFloating }" aria-label="页面导航">
      <a class="brand-mark" href="#hero">CHAIN DEV DOSSIER</a>
      <div class="nav-links">
        <a v-for="item in navItems" :key="item.href" :href="item.href">{{ item.label }}</a>
      </div>
    </nav>

    <section id="hero" class="hero-section">
      <div class="hero-grid">
        <div class="hero-content">
          <p class="hero-kicker">BLOCKCHAIN ENGINEERING / TRUSTED SYSTEMS</p>
          <h1 class="hero-title">
            <span>链上可信系统</span>
            <span>工程记录</span>
          </h1>
          <p class="hero-copy">
            以 Raft 共识、智能合约与工程交付为主线，整理我的区块链项目实践、能力结构与成长路径。
          </p>
          <div class="hero-actions">
            <a href="#projects" class="primary-action">进入项目记录</a>
            <a href="#profile" class="secondary-action">查看开发者档案</a>
          </div>
        </div>

        <div class="network-visual" aria-label="区块链网络动态视觉">
          <canvas ref="networkCanvas" class="network-canvas"></canvas>
          <svg class="network-routes" viewBox="0 0 640 520" aria-hidden="true">
            <path class="route-path main" d="M72 402 C164 318 210 164 332 208 S462 328 568 118" />
            <path class="route-path secondary" d="M86 122 C178 184 246 92 330 156 S474 206 548 382" />
            <path class="route-path accent" d="M124 462 C238 376 294 414 388 318 S500 238 584 268" />
          </svg>
          <div class="network-overlay">
            <div class="chain-stack">
              <article v-for="block in chainBlocks" :key="block.title" class="chain-card">
                <span>{{ block.title }}</span>
                <strong>{{ block.hash }}</strong>
                <p>{{ block.meta }}</p>
              </article>
            </div>
            <div class="tx-stream" aria-hidden="true">
              <span class="tx-line"></span>
              <span class="tx-line"></span>
              <span class="tx-line"></span>
            </div>
            <div class="network-console">
              <span>SYNC STATUS</span>
              <strong>validated / routed / committed</strong>
            </div>
          </div>
        </div>
      </div>

      <div class="hero-stats" aria-label="技术摘要">
        <div v-for="stat in heroStats" :key="stat.label" class="stat-card">
          <span>{{ stat.label }}</span>
          <strong>{{ stat.value }}</strong>
        </div>
      </div>
    </section>

    <section id="profile" class="profile-section reveal">
      <div class="profile-card">
        <p class="section-index">DEVELOPER PROFILE</p>
        <h2>李翔 / Li Xiang</h2>
        <p>区块链工程方向，关注分布式系统、智能合约与可信数据协作场景的工程实现。</p>
      </div>
      <div class="profile-tags" aria-label="个人方向">
        <span>Blockchain Engineering</span>
        <span>Distributed Systems</span>
        <span>Smart Contract</span>
      </div>
    </section>

    <section id="projects" class="content-section">
      <div class="section-heading reveal">
        <p class="section-index">01 / SELECTED WORK</p>
        <h2>项目证据</h2>
        <p>用项目证明能力，而不是只堆技术名词。重点展示分布式系统、智能合约和工程表达三个方向。</p>
      </div>

      <div class="project-workbench reveal">
        <div class="project-tabs" role="tablist" aria-label="项目切换" :style="projectTabStyle">
          <span class="tab-indicator" aria-hidden="true"></span>
          <button
            v-for="(project, index) in projects"
            :id="`project-tab-${project.code}`"
            :key="project.code"
            class="switch-tab"
            :class="{ active: activeProjectIndex === index }"
            type="button"
            role="tab"
            :aria-selected="activeProjectIndex === index"
            :aria-controls="`project-panel-${project.code}`"
            @click="selectProject(index)"
          >
            <span>{{ project.code }}</span>
            <strong>{{ project.label }}</strong>
          </button>
        </div>

        <article
          :id="`project-panel-${activeProject.code}`"
          class="project-focus"
          role="tabpanel"
          :aria-labelledby="`project-tab-${activeProject.code}`"
        >
          <div class="record-head">
            <span>{{ activeProject.label }}</span>
            <strong>{{ activeProject.code }}</strong>
          </div>
          <div class="signal-meter" aria-hidden="true">
            <span v-for="index in 7" :key="index"></span>
          </div>
          <h3>{{ activeProject.title }}</h3>
          <p class="project-summary">{{ activeProject.summary }}</p>
          <div class="role-row">
            <span>ROLE</span>
            <p>{{ activeProject.role }}</p>
          </div>
          <ul class="evidence-list">
            <li v-for="point in activeProject.points" :key="point">{{ point }}</li>
          </ul>
          <div class="tech-row">
            <span v-for="tech in activeProject.tech" :key="tech">{{ tech }}</span>
          </div>
        </article>
      </div>
    </section>

    <section id="skills" class="content-section">
      <div class="section-heading reveal">
        <p class="section-index">02 / CAPABILITY MAP</p>
        <h2>能力结构</h2>
        <p>能力中心不是散乱的工具列表，而是围绕可信系统构建的一组可复用工程能力。</p>
      </div>

      <div class="capability-console reveal">
        <div class="capability-tabs" role="tablist" aria-label="能力方向切换" :style="capabilityTabStyle">
          <span class="tab-indicator" aria-hidden="true"></span>
          <button
            v-for="(pillar, index) in capabilityPillars"
            :id="`capability-tab-${index}`"
            :key="pillar.title"
            class="switch-tab"
            :class="{ active: activeCapabilityIndex === index }"
            type="button"
            role="tab"
            :aria-selected="activeCapabilityIndex === index"
            :aria-controls="`capability-panel-${index}`"
            @click="selectCapability(index)"
          >
            <span>0{{ index + 1 }}</span>
            <strong>{{ pillar.title }}</strong>
          </button>
        </div>

        <article
          :id="`capability-panel-${activeCapabilityIndex}`"
          class="capability-focus"
          role="tabpanel"
          :aria-labelledby="`capability-tab-${activeCapabilityIndex}`"
        >
          <span class="capability-label">ACTIVE CAPABILITY</span>
          <h3>{{ activeCapability.title }}</h3>
          <p>{{ activeCapability.text }}</p>
          <div class="keyword-row">
            <span v-for="keyword in activeCapability.keywords" :key="keyword">{{ keyword }}</span>
          </div>
        </article>

        <div class="skill-matrix">
          <div v-for="group in skillGroups" :key="group.label" class="skill-group">
            <span>{{ group.label }}</span>
            <p>{{ group.items }}</p>
          </div>
        </div>
      </div>
    </section>

    <section id="timeline" class="content-section">
      <div class="section-heading reveal">
        <p class="section-index">03 / GROWTH TRACE</p>
        <h2>成长路径</h2>
        <p>从专业学习到项目实践，逐步形成面向系统设计、链上开发和工程交付的能力方向。</p>
      </div>

      <div class="timeline-list">
        <article v-for="item in timeline" :key="item.title" class="timeline-record reveal">
          <span>{{ item.tag }}</span>
          <h3>{{ item.title }}</h3>
          <p>{{ item.text }}</p>
        </article>
      </div>
    </section>

    <section id="certificates" class="content-section">
      <div class="section-heading reveal">
        <p class="section-index">04 / HONORS</p>
        <h2>荣誉记录</h2>
        <p>保留可快速扫读的获奖记录，为项目能力增加外部证明。</p>
      </div>

      <div class="certificate-grid">
        <article v-for="cert in certificates" :key="cert.title" class="certificate-card reveal">
          <span>{{ cert.date }}</span>
          <h3>{{ cert.title }}</h3>
          <p>{{ cert.meta }}</p>
        </article>
      </div>
    </section>

    <section id="contact" class="contact-section">
      <div class="section-heading reveal">
        <p class="section-index">05 / CONTACT</p>
        <h2>联系通道</h2>
        <p>如果需要进一步了解项目细节、技术经历或合作可能，可以通过以下方式联系。</p>
      </div>

      <div class="contact-records reveal">
        <a
          v-for="item in contacts"
          :key="item.label"
          :href="item.href"
          class="contact-row"
          :target="item.label === 'GITHUB' ? '_blank' : undefined"
          rel="noreferrer"
        >
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
  --bg: #068562;
  --bg-soft: #5CA87C;
  --panel: rgba(1, 63, 74, 0.62);
  --panel-strong: rgba(1, 63, 74, 0.84);
  --text: #F5FBF2;
  --muted: #D8EBDD;
  --quiet: #B7E5BA;
  --line: rgba(183, 229, 186, 0.28);
  --line-strong: rgba(245, 251, 242, 0.58);
  --green: #B7E5BA;
  --cyan: #5CA87C;
  --amber: #013F4A;
  --green-soft: rgba(183, 229, 186, 0.24);
  --cyan-soft: rgba(92, 168, 124, 0.2);
  --amber-soft: rgba(1, 63, 74, 0.24);
  --shadow: rgba(0, 38, 42, 0.38);
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

.module-glow {
  --edge-proximity: 0;
  --cursor-angle: 45deg;
  --pointer-x: 50%;
  --pointer-y: 50%;
  --glow-strength: max(0, calc((var(--edge-proximity) - 24) / 76));
  --glow-color: 116deg 54% 81%;
  --glow-deep: 189deg 97% 15%;
  position: relative;
  isolation: isolate;
  transform: translate3d(0, 0, 0.01px);
}

.module-glow > .module-edge-light {
  position: absolute;
  inset: -24px;
  z-index: 3;
  pointer-events: none;
  border-radius: inherit;
  opacity: var(--glow-strength);
  mix-blend-mode: plus-lighter;
  transition: opacity 0.45s ease;
  mask-image:
    conic-gradient(
      from var(--cursor-angle) at center,
      black 2%,
      transparent 12%,
      transparent 88%,
      black 98%
    );
}

.module-glow > .module-edge-light::before,
.module-glow > .module-edge-light::after {
  content: '';
  position: absolute;
  inset: 24px;
  border-radius: inherit;
  pointer-events: none;
}

.module-glow > .module-edge-light::before {
  box-shadow:
    inset 0 0 0 1px hsl(var(--glow-color) / 0.95),
    inset 0 0 8px hsl(var(--glow-color) / 0.42),
    inset 0 0 26px hsl(var(--glow-color) / 0.24),
    0 0 2px hsl(var(--glow-color) / 0.72),
    0 0 10px hsl(var(--glow-color) / 0.42),
    0 0 28px hsl(var(--glow-color) / 0.24),
    0 0 58px hsl(var(--glow-color) / 0.12);
}

.module-glow > .module-edge-light::after {
  background:
    radial-gradient(circle at var(--pointer-x) var(--pointer-y), rgba(245, 251, 242, 0.28), transparent 34%),
    radial-gradient(at 80% 55%, rgba(183, 229, 186, 0.38), transparent 50%),
    radial-gradient(at 8% 6%, rgba(92, 168, 124, 0.3), transparent 50%),
    radial-gradient(at 86% 85%, rgba(1, 63, 74, 0.24), transparent 50%);
  opacity: 0.54;
  mix-blend-mode: soft-light;
}

.module-glow:hover {
  border-color: rgba(245, 251, 242, 0.5);
}

.module-glow.stat-card,
.module-glow.network-visual,
.module-glow.project-workbench,
.module-glow.capability-console,
.module-glow.project-focus,
.module-glow.capability-focus,
.module-glow.timeline-record,
.module-glow.certificate-card,
.module-glow.skill-matrix,
.module-glow.contact-records,
.module-glow.chain-card,
.module-glow.network-console {
  overflow: visible;
}

.portfolio-shell {
  min-height: 100vh;
  overflow: hidden;
  background:
    linear-gradient(rgba(245, 251, 242, 0.09) 1px, transparent 1px),
    linear-gradient(90deg, rgba(245, 251, 242, 0.07) 1px, transparent 1px),
    radial-gradient(circle at 30% 10%, rgba(183, 229, 186, 0.55), transparent 34%),
    radial-gradient(circle at 86% 18%, rgba(92, 168, 124, 0.42), transparent 28%),
    radial-gradient(circle at 68% 92%, rgba(1, 63, 74, 0.36), transparent 34%),
    linear-gradient(155deg, #5CA87C 0%, #068562 42%, #1A5140 72%, #013F4A 100%);
  background-size: 44px 44px, 44px 44px, auto, auto, auto, auto;
}

.portfolio-shell::before {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  opacity: 0.2;
  background: repeating-linear-gradient(0deg, rgba(255,255,255,0.026) 0 1px, transparent 1px 4px);
  mix-blend-mode: soft-light;
}

.intro-overlay {
  position: fixed;
  inset: 0;
  z-index: 50;
  overflow: hidden;
  color: var(--text);
  background: #068562;
}

.intro-split {
  position: absolute;
  left: 0;
  right: 0;
  height: 50%;
  background:
    radial-gradient(circle at 50% 100%, rgba(183, 229, 186, 0.32), transparent 36%),
    linear-gradient(120deg, #5CA87C, #068562 46%, #013F4A);
  z-index: 1;
  will-change: transform;
}

.intro-split.top {
  top: 0;
  border-bottom: 1px solid rgba(183, 229, 186, 0.34);
}

.intro-split.bottom {
  bottom: 0;
  border-top: 1px solid rgba(92, 168, 124, 0.32);
}

.intro-stage {
  position: relative;
  z-index: 2;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 28px;
  background:
    radial-gradient(circle at 50% 44%, rgba(183, 229, 186, 0.3), transparent 24%),
    radial-gradient(circle at 40% 58%, rgba(92, 168, 124, 0.22), transparent 32%),
    linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0));
  will-change: transform, opacity;
}

.intro-grid {
  position: absolute;
  inset: 14%;
  opacity: 0.42;
  pointer-events: none;
}

.intro-gridline {
  position: absolute;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(183, 229, 186, 0.42), rgba(92, 168, 124, 0.3), transparent);
  will-change: transform;
}

.intro-gridline:nth-child(1) { top: 18%; }
.intro-gridline:nth-child(2) { top: 42%; }
.intro-gridline:nth-child(3) { top: 64%; }
.intro-gridline:nth-child(4) { top: 82%; }

.intro-access-frame {
  position: absolute;
  width: min(720px, 84vw);
  aspect-ratio: 1.45;
  border: 1px solid rgba(183, 229, 186, 0.22);
  border-radius: 8px;
  background:
    linear-gradient(90deg, transparent 49.7%, rgba(183, 229, 186, 0.16) 50%, transparent 50.3%),
    linear-gradient(0deg, transparent 49.7%, rgba(92, 168, 124, 0.15) 50%, transparent 50.3%),
    linear-gradient(135deg, rgba(183, 229, 186, 0.08), transparent 36%, rgba(6, 133, 98, 0.08));
  box-shadow:
    inset 0 0 48px rgba(183, 229, 186, 0.07),
    0 0 80px rgba(6, 133, 98, 0.18);
  pointer-events: none;
  will-change: clip-path, opacity;
}

.intro-access-frame::before,
.intro-access-frame::after {
  content: '';
  position: absolute;
  left: 8%;
  right: 8%;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(183,229,186,0.58), rgba(92,168,124,0.34), transparent);
  filter: drop-shadow(0 0 12px rgba(183,229,186,0.36));
}

.intro-access-frame::before {
  top: 22%;
}

.intro-access-frame::after {
  bottom: 24%;
}

.intro-corner {
  position: absolute;
  width: 54px;
  height: 54px;
  border-color: rgba(183, 229, 186, 0.74);
  filter: drop-shadow(0 0 12px rgba(183,229,186,0.34));
}

.intro-corner.c1 {
  left: -1px;
  top: -1px;
  border-left: 2px solid;
  border-top: 2px solid;
}

.intro-corner.c2 {
  right: -1px;
  top: -1px;
  border-right: 2px solid;
  border-top: 2px solid;
}

.intro-corner.c3 {
  left: -1px;
  bottom: -1px;
  border-left: 2px solid;
  border-bottom: 2px solid;
}

.intro-corner.c4 {
  right: -1px;
  bottom: -1px;
  border-right: 2px solid;
  border-bottom: 2px solid;
}

.intro-orbit {
  position: absolute;
  left: 50%;
  top: 50%;
  width: min(300px, 52vw);
  aspect-ratio: 1;
  transform: translate(-50%, -50%);
  transform-origin: 0 0;
  opacity: 0.82;
}

.intro-orbit span {
  position: absolute;
  inset: calc(var(--ring, 0) * 22px);
  border: 1px solid rgba(92, 168, 124, 0.26);
  border-radius: 50%;
}

.intro-orbit span:nth-child(1) {
  --ring: 0;
  border-top-color: rgba(183, 229, 186, 0.78);
}

.intro-orbit span:nth-child(2) {
  --ring: 1;
  border-right-color: rgba(92, 168, 124, 0.66);
  transform: rotate(28deg);
}

.intro-orbit span:nth-child(3) {
  --ring: 2;
  border-bottom-color: rgba(183, 229, 186, 0.52);
  transform: rotate(-34deg);
}

.intro-biometric {
  position: absolute;
  left: 9%;
  top: 18%;
  width: 120px;
  display: grid;
  gap: 7px;
}

.intro-biometric span {
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--green), rgba(92,168,124,0.36), transparent);
  transform-origin: left center;
}

.intro-biometric span:nth-child(3n) {
  width: 76%;
}

.intro-biometric span:nth-child(4n) {
  width: 58%;
  background: linear-gradient(90deg, var(--amber), rgba(40,135,96,0.18), transparent);
}

.intro-geo-trace {
  position: absolute;
  right: 9%;
  bottom: 18%;
  width: 170px;
  height: 112px;
  border: 1px solid rgba(210, 235, 218, 0.1);
  background:
    linear-gradient(rgba(255,255,255,0.035) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.035) 1px, transparent 1px);
  background-size: 22px 22px;
}

.trace-node {
  position: absolute;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 16px rgba(183,229,186,0.62);
}

.trace-node.a { left: 24px; top: 34px; }
.trace-node.b { left: 86px; top: 58px; background: var(--cyan); }
.trace-node.c { right: 28px; top: 28px; background: var(--amber); }

.trace-path {
  position: absolute;
  height: 1px;
  background: linear-gradient(90deg, rgba(183,229,186,0.18), rgba(183,229,186,0.64), rgba(92,168,124,0.32));
  transform-origin: left center;
}

.trace-path.p1 { left: 31px; top: 41px; width: 72px; transform: rotate(21deg); }
.trace-path.p2 { left: 92px; top: 61px; width: 72px; transform: rotate(-25deg); }

.intro-readout {
  position: absolute;
  left: clamp(18px, 6vw, 86px);
  bottom: clamp(92px, 13vh, 150px);
  z-index: 3;
  display: grid;
  gap: 8px;
  width: min(320px, calc(100vw - 36px));
  margin: 0;
  padding: 0;
  list-style: none;
  pointer-events: none;
}

.intro-readout li {
  display: grid;
  grid-template-columns: 82px 1fr;
  gap: 14px;
  align-items: center;
  min-height: 32px;
  padding: 8px 10px;
  border: 1px solid rgba(210, 235, 218, 0.12);
  border-radius: 6px;
  background: rgba(8, 13, 11, 0.58);
  backdrop-filter: blur(10px);
}

.intro-readout span,
.intro-readout strong,
.intro-number-cloud span {
  font-family: var(--font-mono);
}

.intro-readout span {
  color: var(--quiet);
  font-size: 0.68rem;
}

.intro-readout strong {
  color: var(--green);
  font-size: 0.72rem;
  font-weight: 500;
}

.intro-number-cloud {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
}

.intro-number-cloud span {
  position: absolute;
  color: rgba(238, 244, 239, 0.28);
  font-size: 0.78rem;
  letter-spacing: 0;
}

.intro-number-cloud span:nth-child(1) { left: 18%; top: 23%; }
.intro-number-cloud span:nth-child(2) { right: 20%; top: 26%; color: rgba(92,168,124,0.38); }
.intro-number-cloud span:nth-child(3) { left: 11%; top: 61%; color: rgba(183,229,186,0.36); }
.intro-number-cloud span:nth-child(4) { right: 13%; top: 58%; }
.intro-number-cloud span:nth-child(5) { left: 28%; bottom: 15%; color: rgba(6,133,98,0.42); }
.intro-number-cloud span:nth-child(6) { right: 31%; bottom: 18%; }

.intro-dna {
  position: absolute;
  right: clamp(22px, 8vw, 120px);
  top: clamp(96px, 18vh, 160px);
  z-index: 2;
  display: grid;
  gap: 9px;
  width: 118px;
  pointer-events: none;
}

.intro-dna span {
  position: relative;
  height: 1px;
  background: linear-gradient(90deg, var(--cyan), transparent 46%, var(--green));
  transform: rotate(calc((var(--i, 0) - 4) * 4deg));
}

.intro-dna span::before,
.intro-dna span::after {
  content: '';
  position: absolute;
  top: -3px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 12px rgba(183,229,186,0.52);
}

.intro-dna span::before {
  left: 0;
}

.intro-dna span::after {
  right: 0;
  background: var(--cyan);
  box-shadow: 0 0 12px rgba(92,168,124,0.5);
}

.intro-dna span:nth-child(1) { --i: 1; }
.intro-dna span:nth-child(2) { --i: 2; }
.intro-dna span:nth-child(3) { --i: 3; }
.intro-dna span:nth-child(4) { --i: 4; }
.intro-dna span:nth-child(5) { --i: 5; }
.intro-dna span:nth-child(6) { --i: 6; }
.intro-dna span:nth-child(7) { --i: 7; }
.intro-dna span:nth-child(8) { --i: 8; }

.intro-constellation {
  position: absolute;
  width: min(560px, 76vw);
  aspect-ratio: 1;
  border: 1px solid rgba(210, 235, 218, 0.12);
  border-radius: 50%;
  opacity: 0.88;
  pointer-events: none;
}

.intro-node,
.intro-pulse {
  position: absolute;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 18px rgba(183, 229, 186, 0.68);
  will-change: transform, opacity;
}

.intro-node.n1 { left: 18%; top: 28%; }
.intro-node.n2 { left: 68%; top: 22%; background: var(--cyan); box-shadow: 0 0 18px rgba(92, 168, 124, 0.62); }
.intro-node.n3 { left: 76%; top: 58%; }
.intro-node.n4 { left: 34%; top: 68%; background: var(--amber); box-shadow: 0 0 18px rgba(40, 135, 96, 0.56); }
.intro-node.n5 { left: 50%; top: 44%; }
.intro-node.n6 { left: 22%; top: 52%; background: var(--cyan); box-shadow: 0 0 18px rgba(92, 168, 124, 0.62); }

.intro-link {
  position: absolute;
  height: 1px;
  background: linear-gradient(90deg, rgba(183,229,186,0.05), rgba(183,229,186,0.5), rgba(92,168,124,0.34));
  transform-origin: left center;
}

.intro-link.l1 { left: 22%; top: 31%; width: 50%; transform: rotate(-7deg); }
.intro-link.l2 { left: 52%; top: 46%; width: 31%; transform: rotate(38deg); }
.intro-link.l3 { left: 25%; top: 54%; width: 45%; transform: rotate(22deg); }

.intro-pulse {
  left: 50%;
  top: 44%;
  width: 180px;
  height: 180px;
  margin: -90px 0 0 -90px;
  border: 1px solid rgba(183, 229, 186, 0.24);
  background: transparent;
  box-shadow: 0 0 72px rgba(183, 229, 186, 0.22);
}

.intro-status,
.intro-copy,
.intro-skip,
.hero-kicker,
.section-index,
.record-head,
.stat-card span,
.chain-card span,
.chain-card strong,
.network-console,
.role-row span,
.tech-row span,
.keyword-row span,
.skill-group span,
.switch-tab,
.capability-label,
.timeline-record span,
.certificate-card span,
.contact-row span,
.brand-mark,
.nav-links a {
  font-family: var(--font-mono);
}

.intro-status {
  position: relative;
  margin: 0 0 20px;
  color: var(--green);
  font-size: 0.76rem;
  letter-spacing: 0;
}

.intro-title {
  position: relative;
  display: grid;
  gap: 0.04em;
  margin: 0;
  text-align: center;
  font-size: clamp(3.8rem, 14vw, 10rem);
  line-height: 0.86;
  font-weight: 700;
}

.intro-title-mask {
  display: block;
  overflow: hidden;
}

.intro-title-line {
  display: block;
  will-change: transform, opacity;
}

.intro-title-mask.accent {
  color: var(--green);
  text-shadow: 0 0 30px rgba(183, 229, 186, 0.34);
}

.intro-copy {
  position: relative;
  margin: 22px 0 0;
  color: var(--muted);
  font-size: 0.9rem;
}

.intro-skip {
  position: absolute;
  right: clamp(18px, 4vw, 44px);
  bottom: clamp(18px, 4vw, 44px);
  min-height: 38px;
  padding: 0 16px;
  border: 1px solid rgba(210, 235, 218, 0.2);
  border-radius: 6px;
  color: var(--text);
  background: rgba(255,255,255,0.035);
  cursor: pointer;
}

.site-nav {
  position: fixed;
  inset: 0 0 auto 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px clamp(20px, 4vw, 64px);
  width: auto;
  border: 1px solid transparent;
  border-radius: 0;
  background: linear-gradient(180deg, rgba(1, 63, 74, 0.72), rgba(6, 133, 98, 0.18), rgba(6, 133, 98, 0));
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  box-shadow: none;
  transition:
    inset 0.34s cubic-bezier(.2,.8,.2,1),
    width 0.34s cubic-bezier(.2,.8,.2,1),
    padding 0.34s cubic-bezier(.2,.8,.2,1),
    border-color 0.24s ease,
    border-radius 0.34s cubic-bezier(.2,.8,.2,1),
    background 0.24s ease,
    box-shadow 0.24s ease;
}

.site-nav.is-floating {
  inset: 16px auto auto max(20px, calc((100% - 760px) / 2));
  width: min(760px, calc(100% - 40px));
  padding: 12px 18px;
  border-color: rgba(245, 251, 242, 0.28);
  border-radius: 999px;
  background:
    linear-gradient(135deg, rgba(245, 251, 242, 0.18), rgba(183, 229, 186, 0.08)),
    rgba(1, 63, 74, 0.48);
  backdrop-filter: blur(22px) saturate(150%);
  -webkit-backdrop-filter: blur(22px) saturate(150%);
  box-shadow:
    0 18px 48px rgba(1, 63, 74, 0.28),
    inset 0 1px 0 rgba(245, 251, 242, 0.28),
    inset 0 -1px 0 rgba(1, 63, 74, 0.24);
}

.site-nav.is-floating::before {
  content: '';
  position: absolute;
  inset: 1px;
  pointer-events: none;
  border-radius: inherit;
  background: linear-gradient(180deg, rgba(245, 251, 242, 0.16), transparent 46%);
  opacity: 0.86;
}

.brand-mark {
  color: var(--text);
  text-shadow: 0 0 18px rgba(183, 229, 186, 0.28);
  font-size: 0.82rem;
  text-decoration: none;
  white-space: nowrap;
  transition: color 0.24s ease, text-shadow 0.24s ease;
}

.nav-links {
  display: flex;
  gap: 24px;
}

.nav-links a {
  color: rgba(245, 251, 242, 0.78);
  font-size: 0.86rem;
  text-decoration: none;
  transition: color 0.2s ease;
}

.nav-links a:hover {
  color: var(--green);
}

.site-nav.is-floating .brand-mark,
.site-nav.is-floating .nav-links a {
  position: relative;
  z-index: 1;
}

.site-nav.is-floating .brand-mark {
  color: #ffffff;
}

.hero-section,
.content-section,
.contact-section,
.profile-section {
  position: relative;
  z-index: 2;
  width: min(1180px, calc(100% - 40px));
  margin: 0 auto;
}

.content-section,
.contact-section,
.profile-section {
  content-visibility: auto;
  contain-intrinsic-size: 900px;
}

.hero-section {
  min-height: 100vh;
  padding: 126px 0 44px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.hero-grid {
  display: grid;
  grid-template-columns: minmax(0, 0.95fr) minmax(380px, 1.05fr);
  gap: clamp(36px, 6vw, 76px);
  align-items: center;
  flex: 1;
}

.hero-kicker {
  margin: 0 0 18px;
  color: var(--green);
  font-size: 0.82rem;
}

.hero-title {
  max-width: 760px;
  margin: 0;
  font-size: clamp(3.2rem, 7vw, 6.1rem);
  line-height: 0.98;
  font-weight: 700;
}

.hero-title span {
  display: block;
}

.hero-title span:last-child {
  color: #013F4A;
  text-shadow: 0 16px 46px rgba(1, 63, 74, 0.18);
}

.hero-copy {
  max-width: 680px;
  margin: 28px 0 0;
  color: var(--muted);
  font-size: 1.05rem;
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
  transition: background 0.2s ease, border-color 0.2s ease, color 0.2s ease;
}

.primary-action {
  color: #013F4A;
  background: var(--green);
  border-color: var(--green);
  box-shadow: 0 14px 30px rgba(1, 63, 74, 0.16), inset 0 -1px 0 rgba(1, 63, 74, 0.16);
}

.secondary-action {
  color: var(--text);
  background: rgba(1, 63, 74, 0.18);
  backdrop-filter: blur(10px);
}

.secondary-action:hover {
  color: var(--green);
  border-color: var(--green);
}

.network-visual {
  position: relative;
  min-height: 560px;
  border: 1px solid rgba(245, 251, 242, 0.46);
  border-radius: 8px;
  overflow: hidden;
  background:
    linear-gradient(120deg, rgba(245,251,242,0.16), transparent 36%),
    linear-gradient(135deg, rgba(1,63,74,0.34), rgba(92,168,124,0.18)),
    rgba(1,63,74,0.28);
  box-shadow:
    0 34px 90px rgba(1, 63, 74, 0.34),
    inset 0 0 0 1px rgba(1, 63, 74, 0.26),
    inset 0 -42px 90px rgba(1, 63, 74, 0.16);
  backdrop-filter: blur(10px);
}

.network-visual::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  pointer-events: none;
  box-shadow:
    inset 0 0 0 1px rgba(245, 251, 242, 0.22),
    inset 0 0 80px rgba(1, 63, 74, 0.12);
}

.network-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

.network-routes {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  opacity: 0.78;
  pointer-events: none;
  mix-blend-mode: screen;
}

.route-path {
  fill: none;
  stroke-width: 1.4;
  stroke-linecap: round;
  stroke-dasharray: 10 18;
  animation: route-flow 6.4s linear infinite;
  filter: drop-shadow(0 0 10px rgba(183, 229, 186, 0.28));
}

.route-path.main {
  stroke: rgba(183, 229, 186, 0.6);
}

.route-path.secondary {
  stroke: rgba(92, 168, 124, 0.54);
  stroke-dasharray: 6 20;
  animation-duration: 7.8s;
  animation-direction: reverse;
}

.route-path.accent {
  stroke: rgba(6, 133, 98, 0.46);
  stroke-dasharray: 2 16;
  animation-duration: 5.8s;
}

.network-overlay {
  position: absolute;
  inset: 0;
  padding: clamp(20px, 3vw, 34px);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.chain-stack {
  display: grid;
  gap: 14px;
  max-width: 330px;
  margin-left: auto;
}

.chain-card {
  position: relative;
  padding: 16px 18px;
  border: 1px solid rgba(183, 229, 186, 0.3);
  border-radius: 8px;
  background: rgba(1, 63, 74, 0.76);
  backdrop-filter: blur(14px);
  box-shadow: 0 14px 34px rgba(0,0,0,0.22);
  will-change: transform;
}

.chain-card::before {
  content: '';
  position: absolute;
  left: -1px;
  top: 18px;
  bottom: 18px;
  width: 2px;
  background: var(--green);
  box-shadow: 0 0 16px rgba(183,229,186,0.5);
}

.chain-card span {
  display: block;
  color: var(--quiet);
  font-size: 0.7rem;
}

.chain-card strong {
  display: block;
  margin-top: 8px;
  color: var(--green);
  font-size: 0.9rem;
  font-weight: 500;
}

.chain-card p {
  margin: 8px 0 0;
  color: var(--muted);
  line-height: 1.55;
}

.tx-stream {
  position: relative;
  align-self: stretch;
  height: 150px;
  overflow: hidden;
}

.tx-line {
  position: absolute;
  left: -40%;
  right: 60%;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--green), var(--cyan), transparent);
  box-shadow: 0 0 18px rgba(183,229,186,0.34);
  will-change: transform;
}

.tx-line:nth-child(1) { top: 32px; }
.tx-line:nth-child(2) { top: 74px; opacity: 0.7; }
.tx-line:nth-child(3) { top: 118px; opacity: 0.55; }

.network-console {
  width: fit-content;
  max-width: 100%;
  padding: 14px 16px;
  border: 1px solid rgba(92, 168, 124, 0.32);
  border-radius: 8px;
  background: rgba(1, 63, 74, 0.78);
  backdrop-filter: blur(12px);
}

.network-console span {
  display: block;
  color: var(--cyan);
  font-size: 0.7rem;
  margin-bottom: 7px;
}

.network-console strong {
  color: var(--text);
  font-size: 0.78rem;
  font-weight: 500;
}

.hero-stats {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
  margin-top: 34px;
}

.stat-card {
  position: relative;
  overflow: hidden;
  min-height: 84px;
  padding: 18px 20px;
  border: 1px solid rgba(245, 251, 242, 0.34);
  border-radius: 8px;
  background:
    linear-gradient(135deg, rgba(1, 63, 74, 0.56), rgba(40, 135, 96, 0.42)),
    rgba(1,63,74,0.52);
  backdrop-filter: blur(8px);
  box-shadow:
    0 16px 38px rgba(1, 63, 74, 0.2),
    inset 0 1px 0 rgba(245, 251, 242, 0.18);
}

.stat-card::after {
  content: '';
  position: absolute;
  inset: auto 14px 12px;
  height: 1px;
  background: linear-gradient(90deg, rgba(183,229,186,0.12), rgba(183,229,186,0.52), rgba(92,168,124,0.16));
  transform: scaleX(0.24);
  transform-origin: left center;
  transition: transform 0.32s ease;
}

.stat-card:hover::after {
  transform: scaleX(1);
}

.stat-card span {
  display: block;
  color: rgba(245, 251, 242, 0.78);
  font-size: 0.72rem;
  margin-bottom: 10px;
}

.stat-card strong {
  display: block;
  color: #ffffff;
  font-size: clamp(1rem, 1.7vw, 1.3rem);
  font-weight: 700;
  text-shadow: 0 2px 14px rgba(1, 63, 74, 0.34);
}

.profile-section {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 24px;
  align-items: end;
  padding: 56px 0 22px;
  border-top: 1px solid var(--line);
}

.profile-card h2 {
  margin: 0;
  font-size: clamp(2rem, 5vw, 4.4rem);
  line-height: 1;
}

.profile-card p:last-child {
  max-width: 650px;
  margin: 18px 0 0;
  color: var(--muted);
  line-height: 1.85;
}

.profile-tags {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 8px;
}

.profile-tags span {
  padding: 8px 10px;
  border: 1px solid rgba(40, 135, 96, 0.34);
  border-radius: 6px;
  color: var(--amber);
  background: var(--amber-soft);
  font-family: var(--font-mono);
  font-size: 0.74rem;
}

.content-section,
.contact-section {
  padding: 108px 0;
}

.section-heading {
  max-width: 760px;
  margin-bottom: 42px;
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
}

.section-heading p:last-child {
  margin: 18px 0 0;
  color: var(--muted);
  line-height: 1.9;
}

.project-workbench,
.capability-console,
.project-focus,
.capability-focus,
.timeline-record,
.certificate-card {
  border: 1px solid var(--line);
  border-radius: 8px;
  background: var(--panel);
  box-shadow: 0 22px 70px rgba(1, 63, 74, 0.26);
  backdrop-filter: blur(14px);
  transition: transform 0.22s ease, border-color 0.22s ease, background 0.22s ease;
}

.project-focus:hover,
.capability-focus:hover,
.timeline-record:hover,
.certificate-card:hover {
  transform: translateY(-3px);
  border-color: var(--line-strong);
  background: var(--panel-strong);
}

.project-workbench {
  display: grid;
  grid-template-columns: minmax(0, 1fr);
  gap: 18px;
  padding: 12px;
  background:
    linear-gradient(135deg, rgba(245, 251, 242, 0.18), transparent 40%),
    rgba(1,63,74,0.24);
}

.project-tabs,
.capability-tabs {
  grid-column: 1 / -1;
  position: relative;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 8px;
  padding: 8px;
  border: 1px solid rgba(210, 235, 218, 0.12);
  border-radius: 8px;
  background: rgba(1, 63, 74, 0.34);
  backdrop-filter: blur(10px);
}

.tab-indicator {
  position: absolute;
  z-index: 0;
  left: 8px;
  top: 8px;
  bottom: 8px;
  width: calc((100% - 32px) / 3);
  border: 1px solid rgba(183,229,186,0.34);
  border-radius: 6px;
  background:
    linear-gradient(135deg, rgba(183,229,186,0.18), rgba(92,168,124,0.1)),
    rgba(255,255,255,0.032);
  box-shadow: inset 0 0 0 1px rgba(255,255,255,0.025), 0 12px 30px rgba(0,0,0,0.16);
  transform: translateX(calc(var(--active-tab-index, 0) * (100% + 8px)));
  transition: transform 0.38s cubic-bezier(.2,.8,.2,1), border-color 0.2s ease;
  pointer-events: none;
}

.switch-tab {
  position: relative;
  z-index: 1;
  border: 1px solid transparent;
  border-radius: 6px;
  color: var(--muted);
  background: transparent;
  text-align: left;
  cursor: pointer;
  transition: color 0.2s ease, border-color 0.2s ease, background 0.2s ease, transform 0.2s ease;
}

.switch-tab {
  min-height: 58px;
  padding: 12px 14px;
  overflow: hidden;
}

.switch-tab::before {
  content: '';
  position: absolute;
  inset: 0;
  opacity: 0;
  border-radius: inherit;
  background: linear-gradient(90deg, rgba(183,229,186,0.14), transparent 62%);
  transform: scaleX(0.72);
  transition: opacity 0.24s ease, transform 0.32s cubic-bezier(.2,.8,.2,1);
}

.switch-tab span,
.switch-tab strong {
  position: relative;
  z-index: 1;
  display: block;
}

.switch-tab span {
  color: var(--quiet);
  font-size: 0.7rem;
  margin-bottom: 7px;
}

.switch-tab strong {
  font-size: 0.86rem;
  line-height: 1.45;
  font-weight: 500;
}

.switch-tab:hover,
.switch-tab.active {
  color: var(--text);
  border-color: rgba(183,229,186,0.34);
}

.switch-tab.active::before {
  opacity: 0.7;
  transform: scaleX(1);
}

.project-focus {
  min-height: 460px;
  padding: clamp(24px, 4vw, 40px);
  background:
    linear-gradient(135deg, var(--green-soft), transparent 42%),
    var(--panel);
}

.record-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: var(--quiet);
  font-size: 0.74rem;
  margin-bottom: 26px;
}

.record-head strong {
  color: var(--green);
  font-size: 1.08rem;
  font-weight: 500;
}

.signal-meter {
  display: flex;
  align-items: end;
  gap: 5px;
  width: 92px;
  height: 28px;
  margin: -8px 0 18px;
}

.signal-meter span {
  display: block;
  width: 7px;
  height: 100%;
  min-height: 7px;
  border-radius: 999px;
  background: linear-gradient(180deg, rgba(183,229,186,0.92), rgba(92,168,124,0.28));
  box-shadow: 0 0 14px rgba(183,229,186,0.24);
  transform-origin: bottom center;
}

.signal-meter span:nth-child(2n) {
  background: linear-gradient(180deg, rgba(92,168,124,0.82), rgba(183,229,186,0.18));
}

.project-focus h3,
.capability-focus h3,
.timeline-record h3,
.certificate-card h3 {
  margin: 0;
  font-size: 1.35rem;
  line-height: 1.45;
}

.project-summary {
  margin: 16px 0 0;
  color: var(--muted);
  line-height: 1.85;
}

.role-row {
  display: grid;
  grid-template-columns: 64px 1fr;
  gap: 16px;
  margin-top: 22px;
  padding: 15px 0;
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
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
  display: grid;
  gap: 12px;
  margin: 22px 0 0;
  padding: 0;
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
  top: 0.75em;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--green);
  box-shadow: 0 0 12px rgba(183,229,186,0.48);
}

.tech-row,
.keyword-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 22px;
}

.tech-row span,
.keyword-row span {
  padding: 7px 10px;
  border: 1px solid rgba(183,229,186,0.32);
  border-radius: 6px;
  color: var(--green);
  background: rgba(183,229,186,0.09);
  font-size: 0.74rem;
}

.certificate-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 20px;
}

.certificate-card {
  padding: 28px;
}

.capability-console {
  display: grid;
  grid-template-columns: minmax(280px, 0.72fr) minmax(0, 1fr);
  gap: 18px;
  padding: 12px;
  background:
    linear-gradient(120deg, rgba(245, 251, 242, 0.16), transparent 46%),
    rgba(1,63,74,0.24);
}

.capability-tabs {
  grid-column: 1 / -1;
}

.capability-focus {
  padding: clamp(24px, 4vw, 34px);
  background:
    linear-gradient(135deg, rgba(92,168,124,0.13), transparent 44%),
    var(--panel);
}

.capability-label {
  display: block;
  margin-bottom: 14px;
  color: var(--cyan);
  font-size: 0.72rem;
}

.capability-focus p,
.certificate-card p,
.timeline-record p {
  margin: 14px 0 0;
  color: var(--muted);
  line-height: 1.8;
}

.skill-matrix {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
  background: rgba(1,63,74,0.2);
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

.skill-group span,
.timeline-record span,
.certificate-card span {
  display: block;
  color: var(--green);
  font-size: 0.76rem;
  margin-bottom: 10px;
}

.skill-group p {
  margin: 0;
  color: var(--muted);
  line-height: 1.7;
}

.timeline-list {
  position: relative;
  display: grid;
  gap: 18px;
  max-width: 920px;
}

.timeline-list::before {
  content: '';
  position: absolute;
  left: 10px;
  top: 8px;
  bottom: 8px;
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
  box-shadow: 0 0 16px rgba(183,229,186,0.46);
}

.contact-section {
  padding-bottom: 88px;
}

.contact-records {
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
  background: rgba(1,63,74,0.22);
  backdrop-filter: blur(12px);
}

.contact-row {
  display: grid;
  grid-template-columns: 150px 1fr;
  gap: 20px;
  padding: 22px 24px;
  border-bottom: 1px solid var(--line);
  text-decoration: none;
  transition: background 0.2s ease;
}

.contact-row:last-child {
  border-bottom: 0;
}

.contact-row span {
  color: var(--green);
  font-size: 0.78rem;
}

.contact-row strong {
  font-weight: 500;
  overflow-wrap: anywhere;
}

.contact-row:hover {
  background: rgba(183,229,186,0.08);
}

@keyframes route-flow {
  from {
    stroke-dashoffset: 0;
  }

  to {
    stroke-dashoffset: -112;
  }
}

@media (max-width: 980px) {
  .site-nav.is-floating {
    inset: 14px auto auto max(14px, calc((100% - 320px) / 2));
    width: min(320px, calc(100% - 28px));
    justify-content: center;
    padding: 11px 18px;
  }

  .nav-links {
    display: none;
  }

  .hero-grid,
  .project-workbench,
  .capability-console,
  .certificate-grid,
  .profile-section {
    grid-template-columns: 1fr;
  }

  .profile-tags {
    justify-content: flex-start;
  }

  .network-visual {
    min-height: 480px;
  }

  .hero-stats {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 640px) {
  .hero-section,
  .content-section,
  .contact-section,
  .profile-section {
    width: min(100% - 24px, 1180px);
  }

  .site-nav {
    padding: 14px 12px;
  }

  .site-nav.is-floating {
    inset: 12px auto auto 12px;
    width: calc(100% - 24px);
    min-width: 0;
    padding: 11px 14px;
  }

  .hero-section {
    min-height: auto;
    padding: 90px 0 34px;
  }

  .hero-grid {
    gap: 26px;
  }

  .hero-kicker {
    margin-bottom: 12px;
    font-size: 0.72rem;
  }

  .hero-title {
    font-size: clamp(2.45rem, 12vw, 3.6rem);
    line-height: 1.02;
  }

  .hero-copy,
  .section-heading p:last-child,
  .profile-card p:last-child {
    font-size: 0.95rem;
    line-height: 1.78;
  }

  .hero-copy {
    margin-top: 20px;
  }

  .hero-actions {
    display: grid;
    gap: 10px;
    margin-top: 24px;
  }

  .primary-action,
  .secondary-action {
    width: 100%;
    min-height: 42px;
    padding: 0 14px;
  }

  .content-section,
  .contact-section {
    padding: 72px 0;
  }

  .profile-section {
    padding: 42px 0 12px;
  }

  .section-heading {
    margin-bottom: 28px;
  }

  .section-heading h2 {
    font-size: clamp(1.9rem, 9vw, 2.8rem);
  }

  .profile-card h2 {
    font-size: clamp(2rem, 10vw, 3rem);
  }

  .intro-status {
    max-width: min(320px, 84vw);
    margin-bottom: 14px;
    text-align: center;
    font-size: 0.66rem;
    line-height: 1.55;
  }

  .intro-title {
    font-size: clamp(2.8rem, 16vw, 4.5rem);
  }

  .intro-copy {
    max-width: 270px;
    margin-top: 16px;
    text-align: center;
    font-size: 0.78rem;
  }

  .intro-access-frame {
    top: 50%;
    width: min(430px, 86vw);
    aspect-ratio: 0.82;
    transform: translateY(-50%);
  }

  .intro-orbit {
    width: min(230px, 58vw);
  }

  .intro-constellation {
    width: min(390px, 86vw);
    opacity: 0.5;
  }

  .intro-pulse {
    width: 132px;
    height: 132px;
    margin: -66px 0 0 -66px;
  }

  .intro-readout {
    left: 14px;
    right: 14px;
    bottom: 68px;
    gap: 6px;
    width: auto;
  }

  .intro-readout li {
    grid-template-columns: 58px 1fr;
    min-height: 28px;
    padding: 6px 8px;
  }

  .intro-readout span {
    font-size: 0.62rem;
  }

  .intro-readout strong {
    font-size: 0.66rem;
  }

  .intro-skip {
    right: 14px;
    bottom: 14px;
    min-height: 34px;
    padding: 0 12px;
    font-size: 0.78rem;
  }

  .intro-number-cloud span {
    font-size: 0.64rem;
  }

  .intro-biometric,
  .intro-geo-trace,
  .intro-dna {
    display: none;
  }

  .network-visual {
    min-height: 340px;
  }

  .chain-stack {
    max-width: none;
    gap: 10px;
  }

  .chain-card {
    padding: 12px 14px;
  }

  .chain-card p {
    font-size: 0.86rem;
  }

  .tx-stream {
    height: 92px;
  }

  .network-console {
    width: 100%;
    padding: 12px;
  }

  .hero-stats,
  .project-tabs,
  .capability-tabs,
  .skill-matrix {
    grid-template-columns: 1fr;
  }

  .tab-indicator {
    right: 8px;
    bottom: auto;
    width: auto;
    height: 58px;
    transform: translateY(calc(var(--active-tab-index, 0) * (100% + 8px)));
  }

  .role-row,
  .contact-row {
    grid-template-columns: 1fr;
    gap: 8px;
  }

  .stat-card {
    min-height: 72px;
    padding: 14px;
  }

  .profile-tags {
    gap: 7px;
  }

  .profile-tags span,
  .tech-row span,
  .keyword-row span {
    padding: 6px 8px;
    font-size: 0.68rem;
  }

  .skill-group {
    border-right: 0;
    padding: 16px;
  }

  .skill-group:nth-last-child(2) {
    border-bottom: 1px solid var(--line);
  }

  .project-workbench,
  .capability-console {
    padding: 10px;
    gap: 12px;
  }

  .project-focus,
  .capability-focus,
  .timeline-record,
  .certificate-card {
    min-height: auto;
    padding: 20px;
  }

  .project-focus h3,
  .capability-focus h3,
  .timeline-record h3,
  .certificate-card h3 {
    font-size: 1.12rem;
  }

  .switch-tab {
    min-height: 58px;
    padding: 10px 12px;
  }

  .project-summary,
  .role-row p,
  .evidence-list li,
  .capability-focus p,
  .certificate-card p,
  .timeline-record p,
  .skill-group p {
    font-size: 0.92rem;
    line-height: 1.72;
  }

  .timeline-list {
    gap: 14px;
  }

  .timeline-record {
    margin-left: 24px;
  }

  .timeline-list::before {
    left: 7px;
  }

  .timeline-record::before {
    left: -30px;
    top: 26px;
    width: 11px;
    height: 11px;
  }

  .contact-row {
    padding: 18px;
  }
}

@media (max-width: 380px) {
  .brand-mark {
    font-size: 0.7rem;
  }

  .hero-title {
    font-size: clamp(2.05rem, 11.5vw, 2.8rem);
  }

  .intro-title {
    font-size: clamp(2.45rem, 15vw, 3.5rem);
  }

  .intro-access-frame {
    width: 82vw;
  }

  .intro-readout {
    bottom: 58px;
  }

  .intro-readout li {
    grid-template-columns: 1fr;
    gap: 2px;
  }

  .hero-stats {
    grid-template-columns: 1fr;
  }

  .network-visual {
    min-height: 300px;
  }

  .route-path {
    opacity: 0.62;
  }

  .project-focus,
  .capability-focus,
  .timeline-record,
  .certificate-card {
    padding: 16px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .route-path {
    animation: none;
  }

  .tab-indicator,
  .stat-card::after {
    transition: none;
  }
}
</style>
