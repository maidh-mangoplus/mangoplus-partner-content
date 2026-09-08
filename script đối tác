const API_PUBLIC = 'https://script.google.com/macros/s/AKfycbwqKTYIwGcieaIzhGc2ohMEiC2A0rUm0ZZA2RErJCcdBLmKQOPknkbZYGuGNeBZ1C2Y/exec';

const RANK_CLASS = { 'S+': 'rank-splus', 'S': 'rank-s', 'A+': 'rank-aplus', 'A': 'rank-a' };
const STATUS_CLASS = { 'Mới ra mắt': 'status-new', 'Đang phát sóng': 'status-live' };
const GROUPS = ['Show', 'Phim', 'Short'];
const BAR_CLASS = { 'Show': 'bar-show', 'Phim': 'bar-phim', 'Short': 'bar-short' };

let allData = [];
let state = { group: 'all', excl: false, newOnly: false, search: '' };
let history = [{ view: 'tongquan', focusId: null, scrollY: 0 }];
let historyIndex = 0;

function normalize(str) {
  return (str || '').toString().toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '').replace(/đ/g, 'd');
}
function matches(item) {
  if (state.group !== 'all' && item.nhom !== state.group) return false;
  if (state.excl && !item.docQuyen) return false;
  if (state.newOnly && item.trangThai !== 'Mới ra mắt') return false;
  if (state.search && !normalize(item.ten).includes(state.search)) return false;
  return true;
}

// ---- History (Back/Next) ----
function navigate(entry) {
  history[historyIndex].scrollY = window.scrollY;
  history = history.slice(0, historyIndex + 1);
  history.push(entry);
  historyIndex = history.length - 1;
  applyHistoryEntry(entry, true);
}
function goBack() { if (historyIndex <= 0) return; historyIndex--; applyHistoryEntry(history[historyIndex], false); }
function goNext() { if (historyIndex >= history.length - 1) return; historyIndex++; applyHistoryEntry(history[historyIndex], false); }
function updateNavButtons() {
  document.getElementById('nav-back').disabled = historyIndex <= 0;
  document.getElementById('nav-next').disabled = historyIndex >= history.length - 1;
}
function applyHistoryEntry(entry, isNewNav) {
  switchViewRaw(entry.view);
  if (entry.view === 'chitiet' && entry.focusId) {
    requestAnimationFrame(() => {
      const card = document.querySelector('[data-card-id="' + entry.focusId + '"]');
      if (!card) return;
      card.scrollIntoView({ behavior: 'smooth', block: 'center' });
      card.style.outline = '2px solid var(--brand)';
      setTimeout(() => { card.style.outline = 'none'; }, 1500);
    });
  } else if (!isNewNav) {
    requestAnimationFrame(() => window.scrollTo({ top: entry.scrollY || 0, behavior: 'auto' }));
  }
  updateNavButtons();
}
document.getElementById('nav-back').addEventListener('click', goBack);
document.getElementById('nav-next').addEventListener('click', goNext);

// ---- Chi tiết ----
function pointsHtml(item) {
  if (!item.diemNhan || !item.diemNhan.length) return '';
  return '<div class="points">' + item.diemNhan.map(p => '<div class="point">' + p + '</div>').join('') + '</div>';
}
function posterBannerHtml(item) {
  return '<div class="poster-banner">' +
    (item.docQuyen ? '<div class="excl-badge">Độc quyền</div>' : '') +
    (item.posterNgang ? '<img src="' + item.posterNgang + '" alt="" loading="lazy" onerror="this.remove()">' : '') +
    '</div>';
}
function cardHtml(item) {
  return '<div class="card" data-card-id="' + item.id + '">' +
    posterBannerHtml(item) +
    '<div class="body">' +
      '<div class="tag-row">' +
        (RANK_CLASS[item.hang] ? '<span class="tag ' + RANK_CLASS[item.hang] + '">' + item.hang + '</span>' : '') +
        (STATUS_CLASS[item.trangThai] ? '<span class="tag ' + STATUS_CLASS[item.trangThai] + '">' + item.trangThai + '</span>' : '') +
      '</div>' +
      '<div class="title">' + item.ten + '</div>' +
      '<div class="meta">' + [item.chuyenMuc, item.quocGia, item.namDinhDang].filter(Boolean).join(' · ') + '</div>' +
      '<p class="tomtat" id="tomtat-' + item.id + '">' + (item.tomTat || '') + '</p>' +
      '<p class="mota" id="mota-' + item.id + '" style="display:none">' + (item.moTaChiTiet || item.tomTat || '') + '</p>' +
      pointsHtml(item) +
      '<div class="extra-fields" id="extra-' + item.id + '" style="display:none">' +
        (item.luuY ? '<p class="luuy">' + item.luuY + '</p>' : '') +
        (item.trailerUrl ? '<button class="trailer-btn" data-trailer-id="' + item.id + '">▶ Xem trailer</button>' : '') +
      '</div>' +
      '<button class="more-btn" data-more="' + item.id + '">Xem thêm ▾</button>' +
    '</div></div>';
}
function renderChiTiet() {
  document.getElementById('grid-chitiet').innerHTML = allData.filter(matches).map(cardHtml).join('') ||
    '<div class="state-msg">Không tìm thấy nội dung phù hợp.</div>';
  bindDetailEvents();
  requestAnimationFrame(equalizeBodyHeights);
}
function equalizeBodyHeights() {
  const bodies = document.querySelectorAll('#grid-chitiet .body');
  bodies.forEach(b => b.style.minHeight = '');
  if (window.innerWidth <= 720) return;
  let max = 0;
  bodies.forEach(b => { max = Math.max(max, b.offsetHeight); });
  bodies.forEach(b => { b.style.minHeight = max + 'px'; });
}
function toggleMore(id, forceOpen) {
  const tomtat = document.getElementById('tomtat-' + id);
  const mota = document.getElementById('mota-' + id);
  const extra = document.getElementById('extra-' + id);
  const btn = document.querySelector('[data-more="' + id + '"]');
  const opening = forceOpen || mota.style.display === 'none';
  tomtat.style.display = opening ? 'none' : '';
  mota.style.display = opening ? '' : 'none';
  extra.style.display = opening ? 'block' : 'none';
  if (btn) btn.textContent = opening ? 'Thu gọn ▴' : 'Xem thêm ▾';
}
function bindDetailEvents() {
  document.querySelectorAll('#grid-chitiet [data-more]').forEach(btn => btn.addEventListener('click', () => toggleMore(btn.dataset.more)));
  document.querySelectorAll('#grid-chitiet [data-trailer-id]').forEach(btn => btn.addEventListener('click', () => {
    const item = allData.find(d => String(d.id) === btn.dataset.trailerId);
    if (item) openTrailerModal(item.trailerUrl);
  }));
}

// ---- Tổng quan: Top thịnh hành + lưới nhóm ----
function trendingItemHtml(item, i) {
  const cls = matches(item) ? '' : ' dimmed';
  return '<div class="trending-item' + cls + '" data-nav="' + item.id + '" style="animation-delay:' + (i * 0.05) + 's">' +
    '<div class="trending-poster">' + (item.posterDoc ? '<img src="' + item.posterDoc + '" alt="" loading="lazy" onerror="this.remove()">' : '') + '</div>' +
    '<div class="trending-title">' + item.ten + '</div>' +
    '<div class="trending-usp">' + (item.usp || '') + '</div>' +
  '</div>';
}
function miniCardHtml(item, i) {
  return '<div class="mini-card" data-nav="' + item.id + '" style="animation-delay:' + (i * 0.04) + 's">' +
    '<div class="mini-poster">' +
      (item.docQuyen ? '<div class="excl-badge">Độc quyền</div>' : '') +
      (item.posterDoc ? '<img src="' + item.posterDoc + '" alt="" loading="lazy" onerror="this.remove()">' : '') +
    '</div>' +
    '<div class="mini-title">' + item.ten + '</div>' +
  '</div>';
}
function renderTongQuan() {
  const trending = allData.filter(d => d.usp && d.usp.trim());
  document.getElementById('trendingScroll').innerHTML = trending.map((it, i) => trendingItemHtml(it, i)).join('');
  document.querySelector('.trending-header').style.display = trending.length ? '' : 'none';
  document.querySelector('.trending-wrap').style.display = trending.length ? '' : 'none';

  const filtered = allData.filter(matches);
  const html = GROUPS.map(g => {
    const items = filtered.filter(d => d.nhom === g);
    if (!items.length) return '';
    return '<div class="section-head ' + BAR_CLASS[g] + '"><span>' + g + '</span></div>' +
      '<div class="grid-overview">' + items.map((it, i) => miniCardHtml(it, i)).join('') + '</div>';
  }).join('');
  document.getElementById('overview-groups').innerHTML = html || '<div class="state-msg">Không tìm thấy nội dung phù hợp.</div>';

  document.querySelectorAll('#view-tongquan [data-nav]').forEach(el => {
    el.addEventListener('click', () => navigate({ view: 'chitiet', focusId: parseInt(el.dataset.nav), scrollY: 0 }));
  });
  setTimeout(updateTrendArrows, 50);
}
document.getElementById('trend-prev').addEventListener('click', () => document.getElementById('trendingScroll').scrollBy({ left: -380, behavior: 'smooth' }));
document.getElementById('trend-next').addEventListener('click', () => document.getElementById('trendingScroll').scrollBy({ left: 380, behavior: 'smooth' }));
function updateTrendArrows() {
  const el = document.getElementById('trendingScroll');
  document.getElementById('trend-prev').style.display = el.scrollLeft > 10 ? 'flex' : 'none';
  document.getElementById('trend-next').style.display = (el.scrollLeft + el.clientWidth < el.scrollWidth - 10) ? 'flex' : 'none';
}
document.getElementById('trendingScroll').addEventListener('scroll', updateTrendArrows);
window.addEventListener('resize', updateTrendArrows);

// ---- Trailer modal ----
function extractYoutubeId(url) {
  const m = url.match(/(?:youtu\.be\/|youtube\.com\/(?:watch\?v=|embed\/|shorts\/))([\w-]{11})/);
  return m ? m[1] : null;
}
function extractDriveId(url) { const m = String(url).match(/[-\w]{25,}/); return m ? m[0] : null; }
function openTrailerModal(url) {
  const embedArea = document.getElementById('trailer-embed-area');
  const fallback = document.getElementById('trailer-fallback-link');
  let inner;
  if (!url) { inner = '<div class="trailer-embed">Chưa có trailer</div>'; fallback.style.display = 'none'; }
  else if (/youtube\.com|youtu\.be/.test(url)) {
    const id = extractYoutubeId(url);
    inner = id ? '<iframe class="trailer-iframe" src="https://www.youtube.com/embed/' + id + '" allow="autoplay; encrypted-media" allowfullscreen></iframe>' : '<div class="trailer-embed">Link YouTube không hợp lệ</div>';
    fallback.href = url; fallback.style.display = 'inline-block';
  } else {
    const driveId = extractDriveId(url);
    inner = driveId ? '<iframe class="trailer-iframe" src="https://drive.google.com/file/d/' + driveId + '/preview" allow="autoplay"></iframe>' : '<div class="trailer-embed">Không xem trước được</div>';
    fallback.href = url; fallback.style.display = 'inline-block';
  }
  embedArea.innerHTML = '<div class="trailer-frame">' + inner + '</div>';
  document.getElementById('trailer-modal').style.display = 'flex';
}
function closeTrailerModal() {
  document.getElementById('trailer-modal').style.display = 'none';
  document.getElementById('trailer-embed-area').innerHTML = '';
}
document.getElementById('modal-close').addEventListener('click', closeTrailerModal);
document.getElementById('trailer-modal').addEventListener('click', function (e) { if (e.target === this) closeTrailerModal(); });

// ---- Điều hướng chung ----
function switchViewRaw(view) {
  document.querySelectorAll('.view-tabs button').forEach(b => b.classList.toggle('active', b.dataset.view === view));
  ['tongquan', 'chitiet', 'scheme'].forEach(v => {
    document.getElementById('view-' + v).style.display = v === view ? '' : 'none';
  });
  if (view === 'chitiet') renderChiTiet();
  else renderTongQuan();
}
document.querySelectorAll('.view-tabs button').forEach(b => b.addEventListener('click', () => {
  navigate({ view: b.dataset.view, focusId: null, scrollY: 0 });
}));
document.querySelectorAll('.chip[data-group]').forEach(b => {
  b.addEventListener('click', () => {
    document.querySelectorAll('.chip[data-group]').forEach(x => x.classList.remove('active'));
    b.classList.add('active'); state.group = b.dataset.group;
    refreshCurrentView();
  });
});
document.querySelectorAll('.chip[data-toggle]').forEach(b => {
  b.addEventListener('click', () => {
    b.classList.toggle('active');
    if (b.dataset.toggle === 'excl') state.excl = b.classList.contains('active');
    if (b.dataset.toggle === 'new') state.newOnly = b.classList.contains('active');
    refreshCurrentView();
  });
});
document.getElementById('searchInput').addEventListener('input', function () {
  state.search = normalize(this.value);
  refreshCurrentView();
});
window.addEventListener('resize', () => { if (currentView() === 'chitiet') equalizeBodyHeights(); });
function currentView() { return document.querySelector('.view-tabs button.active').dataset.view; }
function refreshCurrentView() {
  const v = currentView();
  if (v === 'chitiet') renderChiTiet();
  else renderTongQuan();
}

// ---- Khởi tạo ----
async function init() {
  try {
    const res = await fetch(API_PUBLIC);
    if (!res.ok) throw new Error('Network error: ' + res.status);
    const payload = await res.json();
    allData = payload.items.map((d, i) => Object.assign({ id: i + 1 }, d));
    renderTongQuan();
    updateNavButtons();
    if (payload.scheme && payload.scheme.trim()) {
      document.getElementById('schemeContent').innerHTML = payload.scheme;
    } else {
      document.getElementById('schemeContent').innerHTML = '<span style="color:var(--ink-mute)">Chưa có thông báo.</span>';
    }
  } catch (err) {
    document.getElementById('overview-groups').innerHTML = '<div class="state-msg">Không tải được nội dung. Vui lòng thử lại sau.</div>';
    console.error(err);
  }
}
init();
