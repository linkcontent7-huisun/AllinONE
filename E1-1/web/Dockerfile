# 베이스: 공식 nginx 경량 이미지 (과제 방식 A: 웹 서버 베이스 + 정적 콘텐츠 교체)
FROM nginx:alpine
# 커스텀 1: 이미지 이름표(라벨) — 어떤 이미지인지 식별
LABEL org.opencontainers.image.title="e1-1-web"
# 커스텀 2: 환경변수 — 설정과 코드의 분리 연습
ENV APP_ENV=dev
# 커스텀 3: 정적 페이지 교체 — 기본 nginx 첫 화면을 내 페이지로 바꾼다
COPY site/ /usr/share/nginx/html/
EXPOSE 80
