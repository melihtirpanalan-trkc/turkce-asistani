# turkce-asistani
-- ====================================================================
-- TÜRKÇE DERSİ ASİSTANI - SUPABASE VERİTABANI ŞEMASI
-- ====================================================================
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- SINIFLAR TABLOSU
CREATE TABLE IF NOT EXISTS public.siniflar (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    seviye SMALLINT NOT NULL CHECK (seviye IN (5, 6, 7, 8)),
    sube VARCHAR(10) NOT NULL,
    ad VARCHAR(50) GENERATED ALWAYS AS (seviye || '/' || sube) STORED,
    ogretim_yili VARCHAR(20) DEFAULT '2026-2027',
    aciklama TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ÖĞRENCİLER TABLOSU
CREATE TABLE IF NOT EXISTS public.ogrenciler (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    sinif_id UUID REFERENCES public.siniflar(id) ON DELETE CASCADE,
    okul_no INT NOT NULL,
    ad VARCHAR(100) NOT NULL,
    soyad VARCHAR(100) NOT NULL,
    cinsiyet VARCHAR(10) CHECK (cinsiyet IN ('Kız', 'Erkek', 'Diğer')),
    bep_durumu BOOLEAN DEFAULT FALSE,
    ozel_not TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(sinif_id, okul_no)
);

-- KAZANIMLAR TABLOSU
CREATE TABLE IF NOT EXISTS public.kazanimlar (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    seviye SMALLINT NOT NULL CHECK (seviye IN (5, 6, 7, 8)),
    ogrenme_alani VARCHAR(50) NOT NULL,
    kod VARCHAR(30) NOT NULL UNIQUE,
    tanim TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ÖLÇEK DEĞERLENDİRMELERİ (DİNLEME / KONUŞMA)
CREATE TABLE IF NOT EXISTS public.olcek_degerlendirmeleri (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    ogrenci_id UUID REFERENCES public.ogrenciler(id) ON DELETE CASCADE,
    sinav_donemi VARCHAR(50) NOT NULL,
    toplam_puan NUMERIC(5,2) DEFAULT 0,
    ogretmen_yorumu TEXT,
    tarih DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- SORU BANKASI
CREATE TABLE IF NOT EXISTS public.soru_bankasi (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    seviye SMALLINT NOT NULL,
    soru_tipi VARCHAR(30) DEFAULT 'acik_uclu',
    soru_metni TEXT NOT NULL,
    dogru_cevap TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ERİŞİM İZİNLERİ (RLS)
ALTER TABLE public.siniflar ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.ogrenciler ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.kazanimlar ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.olcek_degerlendirmeleri ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.soru_bankasi ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public All Siniflar" ON public.siniflar FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Public All Ogrenciler" ON public.ogrenciler FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Public All Kazanimlar" ON public.kazanimlar FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Public All Olcekler" ON public.olcek_degerlendirmeleri FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Public All Sorular" ON public.soru_bankasi FOR ALL USING (true) WITH CHECK (true);
